This can happen on Solaris machines with a large number of cores. The web container (Jetty) that ORDS embeds to provide Standalone Mode, applies a heuristic for the required ratio of threads to cores and errors if the desired ratio is not met.
 
Out of the box the embedded Jetty in ORDS is configured to have a max of 200 listen threads, you can increase this number by creating a Jetty configuration file as follows:
 
Check where ORDS is configured to store it’s configuration:

Turn on wrap

Copy as text
$ java -jar ords.war configdir
Nov 06, 2015 11:59:15 AM oracle.dbtools.cmdline.ModifyConfigDir execute
INFO: The config.dir value is /tmp/c
Now create a folder within this config.dir location to store the additional standalone jetty configuration (for example using the above example of /tmp/c) :

Turn on wrap

Copy as text
$ mkdir -p /tmp/c/ords/standalone/etc/
Replace /tmp/c with the value indicated by the config.dir value above. So the relative path with the config.dir value is: ords/standalone/etc/.
Now create a file in the above folder with the following contents:

Turn on wrap

Copy as text
<Configure id="Server" class="org.eclipse.jetty.server.Server">
<Get name="ThreadPool">
<Set name="minThreads" type="int">10</Set>
<Set name="maxThreads" type="int">300</Set>
<Set name="idleTimeout" type="int">60000</Set>
<Set name="detailedDump">false</Set>
</Get>
</Configure>
 
Set the maxThreads figure to a figure slightly greater than the value that Jetty reported it needs, for example if Jetty says it needs 274 threads, round that up to 300.
Save this file as maxthreads.xml in the ords/standalone/etc/ folder. 