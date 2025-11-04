# Environment details

**Last tested:** 04-NOV-2025
**ORDS version used:** 25.3.1.r2891312
**Jetty version used:** 12.0.25
**Java version used:** GraalVM EE 21.3.10 for Java 17.0.11



/global/standalone/etc/jetty-access-log.xml
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