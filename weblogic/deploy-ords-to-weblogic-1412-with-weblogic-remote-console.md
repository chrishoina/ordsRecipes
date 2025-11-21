# Environment details

---

**Last tested:** 20-NOV-2025  
**ORDS version:** 25.3.1  
**Oracle database version:** 23.8.0.25.04  
**Jetty version used:** 12.0.25  
**Java version used:** GraalVM EE 21.3.10 (build 17.0.11+7)

---

> &#9998; **Note:** This deployment assumes you have previously installed and configured ORDS for your target database. This tutorial relies on ORDS and the 26ai database in Podman containers to simulate a Development environment. HTTP is used for this example, steps for configuring HTTPS are outside the scope of this guide. APEX installation is outside the scope of this guide.

<!-- &#x270E; <- alternate note HTML entity if above doesn't work.  -->

## Overview

This section describes the process for deploying ORDS, more specifically the `ords.war` file, in Weblogic version 14.1.2. The Weblogic Remote Console is used for uploading the `ords.war` file and deploying the app to a single Admininstrative Server. Managed Server and Clusters are outside the scope of this guide. 

1. Update the `security.verifySSL` ORDS configuration setting to `false` (default is `true`)
2. Create a copy of your ORDS configuration directory
3. Launch your Weblogic Administrative Server
4. Identify a suitable location on the Weblogic Administrative server for the copied ORDS configuration directory
5. Place the copy of your ORDS configuration at the target location (on the Weblogic Server)
6. Log into the Weblogic Adminstrative Server, using the Remote Console
7. Set your Domain's `EnforceValidBasicAuthCredential` security setting to `false`, restart your WebLogic Server
8. Generate a new ords.war file so that it includes the location of the new ORDS configuration directory (where it resides on the WebLogic Server)
9. Upload the updated `ords.war` to your Weblogic Administrative Server's /upload/app directory
10. Deploy the ORDS application
11. Log in to SQL Developer Web or test a well-known ORDS endpoint to verify successful deployment 

### Step 1: Update ORDS configuration

You are encouraged to make a back-up of your ORDS configuration directory (aka `ords_config`) prior to making any changes. Once you have a backup, set ORDS `security.verifySSL` setting to `false`. 

Assuming you have the ORDS `/bin` in your `$PATH`, commands are: 

```sh
ords config set security.verifySSL false # to change from the default (true), to false
ords config list --include-defaults # to review changes, verify the setting was accepted
```

### Step 2: Copy ORDS configuration

Create a temporary (`temp`) directory and copy the contents of the `ords_config` to the new directory. Assuming you are at the root (`temp`) folder, your subdirectories and files may look something like this:

```txt
├── databases
│   └── default
│       ├── pool.xml
│       └── wallet
│           └── cwallet.sso
└── global
    ├── credentials
    └── settings.xml
```

> &#9998; **Note:** In this example, only one local pool (`default`) exists, your structure may differ.

### Step 3: Launch Weblogic

Launch WebLogic. Once the Administration Server is known to be in a `Running` state you will identify an appropriate location for your `ords_config`. 

> &#9998; **Note:** Alternatively, if the `ords_config` is located on the same system as your WebLogic Server, you may omit Steps 4-5.

### Step 4: Find target location for ORDS configuration

In this example, we know (through previous deployments) once the `ords.war` file has been uploaded to WebLogic it will reside at:  
`/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords.war`  
Your location may differ, chose a location that is appropiate for your current set-up. Take note of this location. 

### Step 5: Move ORDS configuration to target

To simplfy this guide, the updated, copied `ords_config` (from [Step 2](#step-2-copy-ords-configuration)) will be moved to the following location:  
`/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords_config`  
Choose a location that makes sense for your current environment. 

> &#9998; **Note:** This should be the *copied version* of the `ords_config`; the version with `security.verifySSL` changed to `false`. This may be an unecessary step depending on whether you have enabled HTTPS. Although HTTPS configuration is outside the scope of this guide.

### Step 6: Launch Weblogic Remote Console

Launch the Weblogic Remote Console. Connect to your **Admin Server Connection Provider**. If one does not exist, select the **Add Admin Server Connection Provider** option and enter your details. In this example the following values are used (your values may vary): 

| **Field** | **Value** | 
| ------------------------ | ----------- |
| Connection Provider Name | AdminServer |
| Use Web Authentication| Disabled (unchecked) |
| Username | [username from your domain.properties file] |
| Password | [password from your domain.properties file] |
| URL      |	https://127.0.0.1:9002 |
| Proxy Override |	n/a |
| Make Insecure Connection | Enabled (checked) | 

### Step 7: Update WebLogic security settings

The **Edit**, **Configuration**, **Monitoring**, and **Security Data** Trees can be accessed from the Console's sidebar or from the landing page. Navigate to the Edit Tree, then click Environment to reveal the dropdown fields. Click Domain. On the Domain page, locate and select (i.e., &#9745; check) **Show Advanced** options. Select the Security tab then disable (toggle off) the **Enforce Valid Basic Auth Credentials** setting. 

Click **Save** (floppy disk drive icon), then click the Shopping Cart. Select **Commit Changes**. An **"One of more servers need to be restarted."** message will appear. Click  **View Servers**. Select (&#9745; check) your Server (AdminServer in this example) followed by Shutdown; choose the most appropriate option. Then Start your server once again. 

While the server is restarting, you can generate a new `ords.war` file for deploying to WebLogic.

---

***A NOTE FOR REVIEWERS: In my actual deployment, I did have ORDS' `security.verifySSL` setting set to `false` BUT, I did have Enforce Valid Basic Auth Credentials set to true, and ORDS still worked. Any ideas why? I thought it wouldn't work this way.***. 

---

### Step 8: Generate new ords.war

> &#9998; **Note:** This step assumes you have an already valid ORDS installation, and the ORDS `/bin` has been added to your `$PATH`.

The `ords.war` file must be updated to include the correct location of the `ords_config` directory. In this example, the new location (on Weblogic) of the `ords_config` directory is at:  
`/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords_config`  
  
  To update the `WEB.XML` file (a WebLogic Deployment Descriptor file) in the `ords.war` file to so it includes the correct `ords_config` location, use the following ORDS CLI command: 

```sh
ords --config /u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords_config war ords2.war 
``` 

The `--config` flag will temporarily override any Environment (`ENV`) settings previously set for the `ords_config` location, allowing you to update the `ords.war` file. When the `ords.war` file regenerates it will indclude the correct *target* `ords_config` location. This example chooses to create an `ords2.war` file so as not to overwrite the existing `ords.war` file. 

> &#9998; **Note:** If the ORDS `/bin` has been added to your `$PATH`, then renaming the new `ords.war` file may not be necessary. The newly created `ords.war` file will be created in whatever current working directory you are in when you issued the `war` command. 

### Step 9: Upload ords.war to WebLogic


### Step 10: Deploy ORDS 
### Step 11: Test/Verify ORDS
