<!-- SOPs -->
<!-- Terminal window 80x24; use "Return to default size"  -->
# How to deploy ORDS to a Weblogic v14.1.2 Administration Server in Development Mode using the WebLogic Remote Console

## Assumptions: 
- ORDS has been installed in your target database
- Weblogic 14.1.2 is installed/configured
  - A single WebLogic Administration Server is accessed via the Remote WebLogic Console
  - WebLogic is deployed in Development Mode

## Environment overview:

### ORDS

- Version used: 25.4.0.r3641739
- ORDS has been installed in a 26ai PDB (`FREEPDB1`) with a default configuration[^1]
- Configured for HTTP traffic
- ORDS `/bin` and `ords_config` are located on `localhost` at the following locations:
    - **ORDS_CONFIG:** `/usr/local/etc/ords/config`
    - **ORDS /bin:** `/usr/local/bin/ords/bin`
- The ORDS `/bin` has been added to your `$PATH`
- The ORDS configuration directory has been set to the `ORDS_CONFIG` environment variable
- A single REST-enabled, `ORDSDEMO` user has been *previously* created

### WebLogic

- Version used: 14.1.2
- Administration Server is accessed via the WebLogic Remote Console v3.0.2[^8]
- Administration Server has been deployed in Development Mode

### Java

JDK used:
- Java(TM) SE Runtime Environment Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79)
- Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79, mixed mode, sharing)

### Database

- Version used: Oracle AI Database 26ai Free Release 23.26.0.0.0[^2]
  - PDB: `FREEPDB1`

## 1. Preparing ORDS

### 1.1 Overview

ORDS has previously been installed in a PDB (`FREEPDB1`) in a containerized Oracle 26ai Database. The ORDS `/bin` and `ords_config` directories have been placed in locations, similar to that of a typical Linux installation: 

  - **ORDS /bin:** `/usr/local/bin/ords/bin`
  - **ORDS_CONFIG:** `/usr/local/etc/ords/config`

### 1.2 Creating temporary ORDS directories

Before deploying ORDS to WebLogic Server, we recommend copying your existing ORDS configuration directory to a location that will be used for your WebLogic pre-deployment tasks.[^3] In this example, the WebLogic `/base_domain` is located at `/Users/Oracle/Middleware/Oracle_Home/user_projects/domains/base_domain`. For demonstration purposes we've copied the contents of the `/usr/local/etc/ords/config` directory to a temporary, related location. 

#### 1.2.1 Creating the new temporary directory**

Copying the ORDS configuration directories will ensure your existing ORDS configuration settings are retained. Copy the directory to a new directory with the following commands: 

```sh
~ % mk dir /Users/Oracle/ords/
~ % cp -R /usr/local/etc/ords/config/ /Users/Oracle/ords/config
```

### 1.3 Disabling security.verifySSL

ORDS will be deployed to a WebLogic Server (`localhost`) running in Development Mode.[^4] When in Development and Production mode, WebLogic Server's SSL/TLS listen port is disabled by default. For this reason, a related ORDS SSL/TLS configuration setting must also be disabled (`security.verifySSL`).

You can disable the `security.verifySSL` configuration setting so it applies to the copied version (of the original ORDS configuration settings) with the following ORDS CLI command:

```sh
ords --config /Users/Oracle/ords/config config set security.verifySSL false 
```

> **Note:** This command allows us to temporarily override the current `ORDS_CONFIG` directory, and instead refer to the ORDS configuration created in [step 1.2.1](#12-creating-temporary-ords-directories). Thus, preserving the ORDS configuration settings of the original ORDS installation.

### 1.4 Generating a new ords.war

Next you will generate a new `ords.war` specifically for WebLogic. This `ords.war` will contain the location of the copied ORDS configuration files (the location that was illustrated in the [previous step](#13-disabling-securityverifyssl)).

The ORDS CLI contains a WAR utility for generating a new `ords.war` such that it appends an additional `config.url` `<context-param>` to the `web.xml` file of the `ords.war`.[^5] Use the following command to generate a new `ords.war` with the correct configuration[^6]:

```sh
ords --config /Users/Oracle/ords/config war /Users/Oracle/ords/ords.war
```

This command: 

  1. adds the `config.url` parameter name with a value of `/Users/Oracle/ords/config` to the `ords.war`'s `web.xml` file
  2. generates a new `ords.war` 
  3. exports it to the target location defined in the final portion of the command. In this example that is: 
  
     `/Users/Oracle/ords/` + `ords.war`

Next you'll perform WebLogic configuration and deployment. 

## 2 Preparing Weblogic

This tutorial:
- assumes a single, Adminstration Server is configured and deployed in Development Mode[^4]
- demonstrates using the WebLogic Remote Console for Adminstration tasks

> **Note:** Some familiarity with WebLogic administration is assumed. 

### 2.2 Accessing the Administration Server

Once you have created an **Admin Server Connection Provider**, you may log in with your WebLogic Administration credentials. 

#### 2.2.1 Updating the Realm Security Model

Prior to deploying your updated `ords.war`, you must update your Realm's Default Security Model. This configuration setting can be found in Realms, by navigating to: **Edit Tree -> Security -> Realms**. 

Choose your Realm from the Security Realms page list view. Under the General tab, update the Security Model Default setting from `DDOnly` (the default) to `CustomRolesAndPolicies`. Save your changes (Save icon at top of the current view). Then click the Shopping Cart Actions icon. You may either: 
  - **View Changes -> Commit Changes** or 
  - from the same view select **Commit Changes**

### 2.3 Deploying the ORDS application

Next, deploy the `ords.war` to your WebLogic Server. Create a new deployment by navigating to: **Edit Tree-> Deployments -> App Deployments -> New**

Enter/Select the following for the App Deployment: 
1. Select a name for the App Deployment (this name will not impact your application's context path; it will remain `/ords`)
2. Select **AdminServer** as the target server
3. Using the Upload dialog, select the filepath for your newly-created `ords.war` (in our example located at `/Users/Oracle/ords/ords.war`)
4. Staging mode: **Default**
5. Security Model: **Custom Roles and Policies**
6. On Deployment: **Start Application**

Once complete, click the **Create** button at the top of the current view. Then click the Shopping Cart Actions icon, and **View Changes -> Commit Changes** or **Commit Changes**.

Once the ORDS application deploys, refer to your WebLogic Server's stdout. You should observe something like: 

```sh
...Mapped local pools from /Users/Oracle/ords/config/databases:
  /ords/                              => default                        => VALID     


2026-02-24T20:08:08.686Z INFO        Oracle REST Data Services initialized
Oracle REST Data Services version : 25.4.0.r3641739
Oracle REST Data Services server info: WebLogic Server 14.1.2.0.0 Tue Nov 26 02:40:45 GMT 2024 2171472 
Oracle REST Data Services java info: Java HotSpot(TM) 64-Bit Server VM Oracle GraalVM 21.0.9+7.1 (build 21.0.9+7-LTS-jvmci-23.1-b79 mixed mode, sharing)
```

> **Note:** Notice how the pools are mapped from the ORDS configuration directory used when you regenerated the `ords.war` in [step 1.4](#14-generating-a-new-ordswar).

## 3 Testing Login

With ORDS fully deployed, you can now access the ORDS landing page. The landing page will be accessible from `http://localhost:7001/ords/_/landing`.[^7] You may now login with your REST-enabled user.

[^1]: An example of a default configuration:

    settings.xml 

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

[^2]: At the time of writing, the latest Oracle 26ai Database container image can be found [here](https://container-registry.oracle.com/ords/ocr/ba/database/free).

[^3]: Copying is recommended as you'll be updating the ORDS configuration settings, in such a way that may impact running ORDS in Standalone mode.

[^4]: [Reference](https://docs.oracle.com/en/middleware/fusion-middleware/weblogic-server/14.1.2/lockd/secure.html#GUID-F980DB67-7CDE-4EF8-986D-D188D4EDB706__GUID-7ED4D85B-3F12-4C9B-80C2-82A9CF05232B) for the various WebLogic Domain Modes and behavior.

[^5]: You can review the new context parameter by creating a duplicate of this new "for WebLogic" ords.war file, and unarchiving it. Locate the `Web.xml` file and you will see the new context parameter name and value, in this example: 

    ```xml
    ...<context-param>
           <param-name>config.url</param-name>
           <param-value>/Users/Oracle/ords/config</param-value>
       </context-param>...
    ```

    Read more about application Context Parameters https://docs.oracle.com/en/middleware/fusion-middleware/weblogic-server/14.1.2/wbapp/web_xml.html#GUID-17556A83-6260-48B0-8D64-4E5EDD1DA431.

[^6]: The use of this command assumes the ORDS `/bin` has been properly set (i.e., added to your `$PATH`). If not, you may execute this command from within your `~/ords/bin` directory.

[^7]: You can also navigate to http://localhost:7001/ords, which will automatically redirect you to http://localhost:7001/ords/_/landing. This is the same behavior for ORDS when deployed in WebLogic, Standalone, or Apache Tomcat.
[^8]: The latest WebLogic Remote Console can be obtained [here](https://github.com/oracle/weblogic-remote-console/releases).

