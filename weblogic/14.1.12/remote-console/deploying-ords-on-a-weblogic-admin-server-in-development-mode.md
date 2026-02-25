<!-- SOPs -->
<!-- Terminal window 80x24; use "Return to default size"  -->
# How to deploy ORDS to a Weblogic v14.1.2 Administration Server in Development Mode using the WebLogic Remote Console

## Assumptions: 
- Assumes you have previously installed ORDS in your target database.
- Weblogic 14.1.2 has been installed
- WebLogic Administration Server is accessed via the Remote WebLogic Console

## Environment overview:

### ORDS

- Version used: 25.4.0.r3641739
- ORDS has been installed in a 26ai PDB (`FREEPDB1`) with a default configuration
- Configured for HTTP traffic
- ORDS `/bin` and `ords_config` are located on `localhost` at the following locations:
    - **ORDS_CONFIG:** `/usr/local/etc/ords/config`
    - **ORDS /bin:** `/usr/local/bin/ords/bin`
- A single REST-enabled, `ORDSDEMO` user has previously been created

### WebLogic

- Version used: 14.1.2
- Administration Server is accessed via the [WebLogic Remote Console v3.0.2](https://github.com/oracle/weblogic-remote-console/releases)
- Administration Server has been deployed in Development Mode

### Java

JDK used:
- Java(TM) SE Runtime Environment Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79)
- Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79, mixed mode, sharing)

### Database

- Version used: Oracle AI Database 26ai Free Release 23.26.0.0.0[^5]

## Preparing ORDS

### Overview

ORDS has been installed in a containerized Oracle 26ai Database. The ORDS `/bin` and `ords_config` directories have been placed in locations similar to that of a typical Linux installation: 

  - **ORDS /bin:** `/usr/local/bin/ords/bin`
  - **ORDS_CONFIG:** `/usr/local/etc/ords/config`

### Updating configuration settings 

Since ORDS will be deployed to a WebLogic Server (running in Development Mode), the ORDS `security.verifySSL` configuration setting will need to be set to `false`.

You can perform this change using the ORDS CLI, using the following command: 

```sh
ords config set security.verifySSL false 
```

### Rebuilding ords.war

You must configure the `ords.war` such that WebLogic can be made aware of the location and contents of ORDS' configuration directory. You have two ways to accomplish this: 

   1. Rebuild the `ords.war` such that it contains the location of the ORDS configuration directory, or
   2. Pass in the location of the ORDS configuration directory as a Java option, prior to starting up the WebLogic Server. 
   
**Option 1** is preferred. The ORDS CLI contains a WAR utility for recreating the `ords.war` such that it appends an additional `<context-param>` to the `web.xml` file of the `ords.war`. 

Prior to modifying your existing `ords.war` you should create a backup. In this example, a backup folder named `backups` has been created, with the backup placed within: `/usr/local/bin/ords/backups/ords.war.backup` (simply add a `.backup` and remove later ths "dummy extension" if you need to use the backup `.war`). 

In order to better organize your various `ords.war` files it is a good idea to create a separate `/weblogic` subdirectory in the `/usr/local/bin/ords/` directory. The examples assume you have done this too.

Next, assuming the ORDS `/bin` has been properly set (i.e., added to your `$PATH`), you can create a new `ords.war` file by executing the following command from your terminal: 

```sh
ords --config /usr/local/etc/ords/config war /usr/local/bin/ords/weblogic/ords.war
```

This command: 

  1. adds the `config.url` parameter name with a value of `/usr/local/etc/ords/config` to the `ords.war`'s `web.xml` file[^2]
  2. rapackages the `ords.war` 
  3. exports it to the location of your choice with the location and name you have provided. In this example: 
  
     `/usr/local/bin/ords/weblogic/` + `ords.war`

Next you'll perform WebLogic configuration and deployment. 

## Preparing Weblogic

This tutorial assumes a single, Adminstration Server is configured and deployed in `Development Mode`.[^3]

This tutorial demonstrates using the WebLogic Remote Console for Adminstration tasks. Some familiarity with WebLogic administration is assumed. 

### Accessing the Admin Server

Once you have created an **Admin Server Connection Provider**, you may log in with your WebLogic Administration credentials. Prior to deploying your new `ords.war` file you must update your Realm's Default Security Model. This configuration setting can be found in the Edit Tree > Security > Realms. 

Choose your Realm from the Security Realms page list view. Under the General tab, update the Security Model Default setting from `DDOnly` (the default) to `CustomRolesAndPolicies` and save your changes (Save icon at top of the current view). Then click the Shopping Cart Actions icon, and either: 
  - **View Changes > Commit Changes** or 
  - from the same view select **Commit Changes**.

### Deploying the ORDS application

Next, you can deploy the `ords.war` to your WebLogic Server.

Edit Tree > Deployments > App Deployments > New

1. Select a name for the App Deployment (this name will not impact your application's context path; it will remain `/ords`)
2. Select AdminServer as the target server
3. Using the Upload dialog, select the filepath for your newly-created ords.war (in our example located at `/usr/local/bin/ords/weblogic/ords.war`)
4. Staging mode: `Default`
5. Security Model: `Custom Roles and Policies`
6. On Deployment: `Start Application`

Once complete, click the `Create` button at the top of the current view. Once again, click the Shopping Cart Actions icon, and either `View Changes > Commit Changes` or simply `Commit Changes`.

After some time, the changes will take effect, you will see this reflected in your WebLogic Server's stdout. You should observe something like: 

```sh

...Mapped local pools from /usr/local/etc/ords/config/databases:
  /ords/                              => default                        => VALID     


2026-02-24T20:08:08.686Z INFO        Oracle REST Data Services initialized
Oracle REST Data Services version : 25.4.0.r3641739
Oracle REST Data Services server info: WebLogic Server 14.1.2.0.0 Tue Nov 26 02:40:45 GMT 2024 2171472 
Oracle REST Data Services java info: Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79 mixed mode, sharing)
```

### Testing Login

The ORDS landing page should now be accessible from `http://localhost:7001/ords/_/landing`.[^4] You may now login as REST-enabled user.

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

    pool.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
    <properties>
    <comment>Saved on Tue Feb 24 17:23:07 UTC 2026</comment>
    <entry key="db.connectionType">basic</entry>
    <entry key="db.hostname">localhost</entry>
    <entry key="db.port">1521</entry>
    <entry key="db.servicename">FREEPDB1</entry>
    <entry key="db.username">ORDS_PUBLIC_USER</entry>
    <entry key="feature.sdw">true</entry>
    <entry key="restEnabledSql.active">true</entry>
    <entry key="security.requestValidationFunction">ords_util.authorize_plsql_gateway</entry>
    </properties>
    ```

[^2]: You can review the new context parameter by creating a duplicate of this new "for WebLogic" ords.war file, and unarchiving it. Locate the `Web.xml` file and you will see the new context parameter name and value, in this example: 

    ```xml
    ...<context-param>
           <param-name>config.url</param-name>
           <param-value>/usr/local/etc/ords/config</param-value>
       </context-param>...
    ```

    Read more about application Context Parameters https://docs.oracle.com/en/middleware/fusion-middleware/weblogic-server/14.1.2/wbapp/web_xml.html#GUID-17556A83-6260-48B0-8D64-4E5EDD1DA431.

[^3]: [Reference](https://docs.oracle.com/en/middleware/standalone/weblogic-server/15.1.1/lockd/secure.html#GUID-F980DB67-7CDE-4EF8-986D-D188D4EDB706) for the various WebLogic Domain Modes and behavior.

[^4]: You can also navigate to http://localhost:7001/ords, which will automatically redirect you to http://localhost:7001/ords/_/landing. This is the same behavior for ORDS when deployed in WebLogic, Standalone, or Apache Tomcat.

[^5]: At the time of writing, the latest Oracle 26ai Database container image can be found [here](https://container-registry.oracle.com/ords/ocr/ba/database/free).
