# Environment details

**Last tested:** 04-NOV-2025
**ORDS version used:** 25.3.1.r2891312
**Jetty version used:** 12.0.25
**Java version used:** GraalVM EE 21.3.10 for Java 17.0.11

## How to create custom Access Logs for ORDS Standalone (Jetty)

To enable GZip compression in ORDS, add this file to your `/ords/config/global/standalone/etc` directory, and restart your ORDS instance.[^1] If you do not have a `/ords/config/global/standalone/etc` directory `cd` to the `standalone` directory, and then `mkdir etc`, then add the file -- in this example named `jetty-access-log.xml`.

### Sample file 

<?xml version="1.0"?>
<!DOCTYPE Configure PUBLIC "-//Jetty//Configure//EN" "http://www.eclipse.org/jetty/configure.dtd">
<Configure id="Server" class="org.eclipse.jetty.server.Server">
            <Set name="requestLog">
              <New id="RequestLogImpl" class="org.eclipse.jetty.server.CustomRequestLog">
                <Arg>/ords/ords-access.log</Arg>
                <Arg>%{remote}a - %u %t "%r" %s %O "%{Referer}i" "%{User-Agent}i"</Arg>
              </New>
            </Set>
</Configure>