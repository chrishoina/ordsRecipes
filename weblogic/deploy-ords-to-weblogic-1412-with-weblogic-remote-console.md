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

This section describes the process for deploying ORDS, more specifically the `ords.war` file, in WebLogic version 14.1.2. The WebLogic Remote Console is used for uploading the `ords.war` file and deploying the app to a single Admininstrative Server. Managed Server and Clusters are outside the scope of this guide. 

1. Update the `security.verifySSL` ORDS configuration setting to `false` (default is `true`)
2. Create a copy of your ORDS configuration directory
3. Launch your WebLogic Administrative Server
4. Identify a suitable location on the WebLogic Administrative server for the copied ORDS configuration directory
5. Place the copy of your ORDS configuration at the target location (on the WebLogic Server)
6. Log into the WebLogic Adminstrative Server, using the Remote Console
7. Set your Domain's `EnforceValidBasicAuthCredential` security setting to `false`, restart your WebLogic Server
8. Generate a new `ords.war` file so that it includes the location of the new ORDS configuration directory (where it resides on the WebLogic Server)
9. Upload the updated `ords.war` to your WebLogic Administrative Server's `/upload/app` directory
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

### Step 3: Launch WebLogic

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

### Step 6: Launch WebLogic Remote Console

Launch the WebLogic Remote Console. Connect to your **Admin Server Connection Provider**. If one does not exist, select the **Add Admin Server Connection Provider** option from the Providers drawer and enter your details. In this example the following values are used (your values may vary): 

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

The **Edit**, **Configuration**, **Monitoring**, and **Security Data** Trees (a.k.a. *Perspectives*) can be accessed from the Console's sidebar or from the landing page. Navigate to the Edit Tree, then click **Environment** to reveal the dropdown fields. Click **Domain**.  

On the Domain page, locate and select (i.e., &#9745; check) **Show Advanced** options. Select the **Security** tab and disable (toggle off) the **Enforce Valid Basic Auth Credentials** setting.  
  
Click **Save** (floppy disk drive icon), then click the Shopping Cart. Select **Commit Changes**. A **"One of more servers need to be restarted."** message will appear. Click  **View Servers**. Select (&#9745; check) your Server (AdminServer in this example) followed by Shutdown; choose the most appropriate option.  
  
Start your server once again. While the server is restarting, you can generate a new `ords.war` file for deploying to WebLogic.

---

**A NOTE FOR REVIEWERS:** In my actual deployment, I did have ORDS' `security.verifySSL` setting set to `false` BUT, I did have Enforce Valid Basic Auth Credentials set to true, and ORDS still worked. Any ideas why? I thought it wouldn't work this way. 

---

### Step 8: Generate new ords.war

> &#9998; **Note:** This step assumes you have an already valid ORDS installation, and the ORDS `/bin` has been added to your `$PATH`. You **DO NOT** need to be in either the `/bin` or `/ords_config` directories to execute this command.

The `ords.war` file must be updated to include the correct location of the `ords_config` directory. In this example, the new location (on WebLogic) of the `ords_config` directory is at:  
`/u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords_config`  
  
To update the `ords.war` file (more specifically, the `WEB.XML`, a WebLogic Deployment Descriptor file, contained within the `ords.war` file) with the correct `ords_config` location, use the following ORDS CLI command (perform this action while in any directory **other than** the ORDS `/bin` and `ords_config` directories): 

    ```sh
    ords --config /u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords_config war ords2.war 
    ``` 

The `--config` flag will temporarily override any Environment (`ENV`) settings previously set for the `ords_config` location, allowing you to generate a new `ords.war` file. When the `ords.war` file regenerates it will indclude the correct *target* `ords_config` location (the `ords_config` as it resides on WebLogic).

### Step 9: Upload ords.war to WebLogic

By this time your WebLogic Administrative server should be back up and running (e.g. you will observe a `<Notice> <WebLogicServer> <BEA-000365> <Server state changed to RUNNING.>` message in your WebLogic's stdout).  

From the WebLogic Remote Console's **Edit Tree** navigate to **Deployments**, then **App Deployments**. Select the **New** button, and enter in your app details. For this example, the following values and selections are used: 

| **Field** | **Value** | 
| ------------------------ | ----------- |
| Name  | `ords` | 
| Targets | AdminServer |
| Upload | Enabled (toggle on) |
| Source | `ords.war` (the newly generated version) |
| Plan | n/a |
| Staging Mode | Default |
| Security Model | Custom Roles |
| On Deployment | Do not start application | 

Once all fields are entered click **&#8853; Create**. Click the Shopping Cart, select **Commit Changes**.  
  
---

**A NOTE FOR REVIEWERS:** The available options for Security Model are below. I use ***Custom Roles*** because I do not see any Roles or Policies in either the WEB.xml or WEBLOGIC.xml files. With that in mind, Custom Roles seems to be the best option. ***Is this correct?***

- DD Only: Use only roles and policies that are defined in the deployment descriptors.
- Custom Roles: Use roles that are defined in the WebLogic Remote Console; use policies that are defined in the deployment descriptor.
- Custom Roles and Policies: Use only roles and policies that are defined in the WebLogic Remote Console.
- Advanced: Use a custom model that you have configured on the realm's configuration page.***

---

### Step 10: Deploy ORDS 

Navigate to the Monitoring Tree. Click Deployments, then Application Management. An *Installed Applications* page will appear, select (&#9745;) `ords` from the list. Click Start, then the Servicing all requests option.

A "Started and created task for application ords. Track progress under Monitoring Tree -> Deployment -> Deployment Tasks" message will appear. You can navigate to this page to view the application's deployment progress. 

### Step 11: Test/Verify ORDS

From the WebLogic stdout, you will observe the ORDS initialization. Messages including the following will appear: 

    ```sh
    2025-11-22T12:28:38.172Z INFO        Created Pool: |default|lo|-2025-11-22T12-28-37.198503769Z at: 2025-11-22T12:28:37.198503769Z
    2025-11-22T12:28:38.404Z INFO        

    Mapped local pools from /u01/oracle/user_projects/domains/base_domain/servers/AdminServer/upload/ords/app/ords_config/databases:
    /ords/                              => default                        => VALID     


    2025-11-22T12:28:38.419Z INFO        Oracle REST Data Services initialized
    Oracle REST Data Services version : 25.3.1.r2891312
    Oracle REST Data Services server info: WebLogic Server 14.1.2.0.0 Tue Nov 26 02:40:45 GMT 2024 2171472 
    Oracle REST Data Services java info: Java HotSpot(TM) 64-Bit Server VM  (build 21.0.7+8-LTS-245 mixed mode, sharing)
    ```

You can now navigate to your server and port (7001 for non-Administrative HTTP/HTTPS traffic). In this example navigating to `localhost:7001/ords` will redirect to the ORDS Landing Page (`/ords/_landing`). You may log in with a REST-enabled user. Once logged in, you will have officialy confirmed that ORDS has successfully deployed.

---
**A NOTE FOR REVIEWERS:** 

When logging into SQL Dev Web I observed:

- 404 Not Found, URL: `http://localhost:7001/ords/admin/_/public-properties/`  
  I do not know if this is expected. I do not know what this endpoint is for. May need further investigation. If anything, we might want to include a brief explanation if a 404 is expected, and why. 

- 401 Not Authorized, URL: `http://localhost:7001/ords/ordstest/_sdw/_services/dba/self_service_schema/requests/availability/`  
  I'm also not sure if this is expected. I vaugely remember seeing something in our docs about self_service_schema, but haven't looked yet. Regardless, here is the complete stack trace (click to expand):
  
    <details>
    <summary><code>"message": "Unauthorized"</code></summary>
    <code>{
        "code": "Unauthorized",
        "message": "Unauthorized",
        "type": "tag:oracle.com,2020:error/Unauthorized",
        "instance": "tag:oracle.com,2020:ecid/oLbZ0b5ebYR00a16j7Rq4g",
        "diagnosticTrace": "",
        "stackTrace": "UnauthorizedException [statusCode=401, logLevel=FINER, reasons=[]]\n\tat oracle.dbtools.http.auth.RequestAuthorizationProvider.authorize(RequestAuthorizationProvider.java:171)\n\tat oracle.dbtools.http.auth.AuthorizationDispatchHook.before(AuthorizationDispatchHook.java:40)\n\tat oracle.dbtools.http.dispatch.hooks.DispatchHookChain.before(DispatchHookChain.java:35)\n\tat oracle.dbtools.http.dispatch.hooks.DispatchHooks.before(DispatchHooks.java:49)\n\tat oracle.dbtools.http.entrypoint.Dispatcher.dispatch(Dispatcher.java:122)\n\tat oracle.dbtools.http.entrypoint.EntryPoint$FilteredServlet.service(EntryPoint.java:166)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:73)\n\tat oracle.dbtools.rest.resource.EnvoyPreDispatchFilter.doFilter(EnvoyPreDispatchFilter.java:124)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.forwarding.QueryFilteringRewrite.doFilter(QueryFilteringRewrite.java:90)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.forwarding.ForwardingFilter.doFilter(ForwardingFilter.java:69)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.cors.CORSPreflightFilter.doFilter(CORSPreflightFilter.java:68)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.cookies.auth.CookieSessionCSRFFilter.doFilter(CookieSessionCSRFFilter.java:69)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.auth.AuthenticationFilter.authenticate(AuthenticationFilter.java:131)\n\tat oracle.dbtools.http.auth.AuthenticationFilter.doFilter(AuthenticationFilter.java:71)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.url.mapping.RequestMapperImpl.doFilter(RequestMapperImpl.java:161)\n\tat oracle.dbtools.url.mapping.URLMappingBase.doFilter(URLMappingBase.java:90)\n\tat oracle.dbtools.url.mapping.db.DatabaseTenantMapping.dispatchSelf(DatabaseTenantMapping.java:216)\n\tat oracle.dbtools.url.mapping.db.DatabaseTenantMappingBase.doFilter(DatabaseTenantMappingBase.java:51)\n\tat oracle.dbtools.url.mapping.tenant.TenantMappingDispatcher.dispatch(TenantMappingDispatcher.java:59)\n\tat oracle.dbtools.url.mapping.db.DatabaseTenantMappingBase.dispatchChild(DatabaseTenantMappingBase.java:152)\n\tat oracle.dbtools.url.mapping.db.DatabaseTenantMappingBase.doFilter(DatabaseTenantMappingBase.java:49)\n\tat oracle.dbtools.url.mapping.tenant.TenantMappingDispatcher.dispatch(TenantMappingDispatcher.java:59)\n\tat oracle.dbtools.jdbc.pools.local.DefaultLocalTenantMapping.dispatchSelf(DefaultLocalTenantMapping.java:156)\n\tat oracle.dbtools.url.mapping.db.DatabaseTenantMappingBase.doFilter(DatabaseTenantMappingBase.java:51)\n\tat oracle.dbtools.jdbc.pools.local.DefaultLocalTenantMapping.doFilter(DefaultLocalTenantMapping.java:112)\n\tat oracle.dbtools.url.mapping.tenant.TenantMappingDispatcher.dispatch(TenantMappingDispatcher.java:59)\n\tat oracle.dbtools.url.mapping.tenant.TenantMappingFilter.doFilter(TenantMappingFilter.java:91)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.forwarding.ForwardingFailedFilter.doFilter(ForwardingFailedFilter.java:42)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.auth.external.ExternalSessionFilter.doFilter(ExternalSessionFilter.java:59)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.apex.support.auth.ApexSessionQueryRewriteFilter.doFilter(ApexSessionQueryRewriteFilter.java:58)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.cors.CORSResponseFilter.doFilter(CORSResponseFilter.java:90)\n\tat oracle.dbtools.http.filters.HttpResponseFilter.doFilter(HttpResponseFilter.java:45)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.filters.AbsoluteLocationFilter.doFilter(AbsoluteLocationFilter.java:65)\n\tat oracle.dbtools.http.filters.HttpResponseFilter.doFilter(HttpResponseFilter.java:45)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.auth.external.ExternalAccessValidationFilter.doFilter(ExternalAccessValidationFilter.java:59)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.errors.ErrorPageFilter.doFilter(ErrorPageFilter.java:87)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.secure.ForceHttpsFilter.doFilter(ForceHttpsFilter.java:74)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.auth.ForceAuthFilter.doFilter(ForceAuthFilter.java:44)\n\tat oracle.dbtools.http.filters.HttpFilter.doFilter(HttpFilter.java:47)\n\tat oracle.dbtools.http.filters.FilterChainImpl.doFilter(FilterChainImpl.java:64)\n\tat oracle.dbtools.http.filters.Filters.filter(Filters.java:67)\n\tat oracle.dbtools.http.entrypoint.EntryPoint.service(EntryPoint.java:69)\n\tat oracle.dbtools.http.entrypoint.EntryPointServlet.service(EntryPointServlet.java:130)\n\tat oracle.dbtools.entrypoint.WebApplicationRequestEntryPoint.service(WebApplicationRequestEntryPoint.java:50)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:750)\n\tat weblogic.servlet.internal.StubSecurityHelper$ServletServiceAction.run(StubSecurityHelper.java:295)\n\tat weblogic.servlet.internal.StubSecurityHelper$ServletServiceAction.run(StubSecurityHelper.java:260)\n\tat weblogic.servlet.internal.StubSecurityHelper.invokeServlet(StubSecurityHelper.java:137)\n\tat weblogic.servlet.internal.ServletStubImpl.execute(ServletStubImpl.java:353)\n\tat weblogic.servlet.internal.TailFilter.doFilter(TailFilter.java:25)\n\tat weblogic.servlet.internal.FilterChainImpl.doFilter(FilterChainImpl.java:87)\n\tat weblogic.servlet.internal.RequestEventsFilter.doFilter(RequestEventsFilter.java:46)\n\tat weblogic.servlet.internal.FilterChainImpl.doFilter(FilterChainImpl.java:87)\n\tat weblogic.servlet.internal.WebAppServletContext$ServletInvocationAction.wrapRun(WebAppServletContext.java:3895)\n\tat weblogic.servlet.internal.WebAppServletContext$ServletInvocationAction.run(WebAppServletContext.java:3855)\n\tat weblogic.security.acl.internal.AuthenticatedSubject.doAs(AuthenticatedSubject.java:344)\n\tat weblogic.security.service.SecurityManager.runAsForUserCode(SecurityManager.java:198)\n\tat weblogic.servlet.provider.WlsSecurityProvider.runAsForUserCode(WlsSecurityProvider.java:203)\n\tat weblogic.servlet.provider.WlsSubjectHandle.run(WlsSubjectHandle.java:71)\n\tat weblogic.servlet.internal.WebAppServletContext.processSecuredExecute(WebAppServletContext.java:2522)\n\tat weblogic.servlet.internal.WebAppServletContext.doSecuredExecute(WebAppServletContext.java:2371)\n\tat weblogic.servlet.internal.WebAppServletContext.securedExecute(WebAppServletContext.java:2346)\n\tat weblogic.servlet.internal.WebAppServletContext.execute(WebAppServletContext.java:2324)\n\tat weblogic.servlet.internal.ServletRequestImpl.runInternal(ServletRequestImpl.java:1837)\n\tat weblogic.servlet.internal.ServletRequestImpl.run(ServletRequestImpl.java:1783)\n\tat weblogic.servlet.provider.ContainerSupportProviderImpl$WlsRequestExecutor.run(ContainerSupportProviderImpl.java:287)\n\tat weblogic.invocation.ComponentInvocationContextManager._runAs(ComponentInvocationContextManager.java:352)\n\tat weblogic.invocation.ComponentInvocationContextManager.runAs(ComponentInvocationContextManager.java:337)\n\tat weblogic.work.LivePartitionUtility.doRunWorkUnderContext(LivePartitionUtility.java:60)\n\tat weblogic.work.PartitionUtility.runWorkUnderContext(PartitionUtility.java:41)\n\tat weblogic.work.SelfTuningWorkManagerImpl.runWorkUnderContext(SelfTuningWorkManagerImpl.java:695)\n\tat weblogic.work.ExecuteThread.execute(ExecuteThread.java:430)\n\tat weblogic.work.ExecuteThread.run(ExecuteThread.java:370)\nCaused by: NotAuthorizedException [authConstraint=oracle.dbtools.sdw.admin, error=null]\n\tat oracle.dbtools.http.auth.RequestAuthorizationProvider.authorize(RequestAuthorizationProvider.java:156)\n\t... 100 more\n"
    }</code>
    </details>
  
- 404 Not Found, URL: `http://localhost:7001/ords/ordstest/_/graphiql/status?q=%7B%7D`  
  **Message:** `message": "ORDS is not running on GraalVM or the JavaScript component is not installed. Unable to start the GraphQL Feature."` I'm wondering if this should flash on screen somewhere, to inform the user? And what the implications are. This could cause issues when starting in standalone and migrating to JaveEE (where Graal is not used), because we known that the docker/podman containers have graalVM with the JS component (so, we don't see that message when deploying in standalone).

  - What is the function of this endpoint? URL: `http://localhost:7001/ords/ordstest/_sdw/_services/user/capabilities/roles`. The response is:

    ```json
    {
      "roles": "SODA_APP" 
    }
    ```

- Is there a way to access this via curl? `http://localhost:7001/ords/ordstest/_sdw/_services/whoami/` provides some excellent details (that could be of use to our support engineers). I've tried uncessfully using basic auth and the `--header 'WWW-Authenticate: Basic realm="weblogic"' ` header. Response:

    ```json
    {
        "username": "ordstest",
        "schema": "ORDSTEST",
        "url-mapping": "ordstest",
        "ords-version": "25.3.1.r2891312",
        "ords-schema-present": true,
        "ords-schema-version": "25.3.1.r2891312",
        "db-major-version": 23,
        "db-minor-version": 8,
        "db-is-cdb": false,
        "user-admin": false,
        "cloud-service": "",
        "ords-context-name": "/ords"
    }
    ```

 

---