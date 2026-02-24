<!-- SOPs -->
<!-- Terminal window 80x24; use "Return to default size"  -->
# How to deploy ORDS to a Weblogic v14.1.2 Administration Server in Development Mode

## Assumptions: 
Assumes you have previously installed ORDS in your target database
Weblogic 14.1.2 has been installed 

## Environment overview:

### ORDS

- Version used: 25.4.0.r3641739
- ORDS has been installed in a 26ai PDB (`FREEPDB1`)with a default configuration[^1]
- Configured for HTTP traffic
- ORDS `/bin` and `ords_config` are located on `localhost` at the following locations:
    - **ORDS_CONFIG:** `/usr/local/etc/ords/config`
    - **ORDS /bin:** `/usr/local/bin/ords/bin`
- A single REST-enabled, `ORDSDEMO` user has previously been created

x


### WebLogic

Version used: 14.1.2
Administration Server is accessed via the WebLogic Remote Console v3.0.2
Administration Server has been deplopyed in Development Mode

### Java

JDK used:
- Java(TM) SE Runtime Environment Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79)
- Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79, mixed mode, sharing)

### Database

- Version used: Oracle AI Database 26ai Free Release 23.26.0.0.0

[^1]: settings.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
    <properties>
    <comment>Saved on Tue Feb 24 17:23:07 UTC 2026</comment>
    <entry key="database.api.enabled">true</entry>
    <entry key="standalone.doc.root">/usr/local/etc/ords/config/global/doc_root</entry>
    <entry key="standalone.http.port">8080</entry>
    </properties>
    ```