This section consists of the following blocks that are related to the tuning and debugging/troubleshooting:

- [JVM tuning for clusters with ON_HEAP caches](doc:jvm-and-system-tuning#recommendations-for-clusters-with-on_heap-caches)
- [JVM tuning for clusters with OFF_HEAP caches](doc:jvm-and-system-tuning#recommendations-for-clusters-with-off_heap-caches)
- [GC attacks by Linux](doc:jvm-and-system-tuning#gc-attacks-by-linux)
- [Getting heap dump on OutOfMemory errors](doc:jvm-and-system-tuning#getting-heap-dump-on-out-of-memory-errors)
- [Detailed garbage collection stats](doc:jvm-and-system-tuning#detailed-garbage-collection-stats)
- [FlighRecorder settings](doc:jvm-and-system-tuning#flightrecorder-settings)
[block:api-header]
{
  "type": "basic",
  "title": "Recommendations for clusters with ON_HEAP caches"
}
[/block]
This section contains basic recommendations for clusters that keeps all or significant amount of cache entries in Java heap. 

##Basic JVM configuration

The following JVM settings have proven to provide fairly smooth throughput without large spikes:
[block:code]
{
  "codes": [
    {
      "code": "-server\n-XX:+UseParNewGC \n-XX:+UseConcMarkSweepGC \n-XX:+UseTLAB \n-XX:NewSize=128m \n-XX:MaxNewSize=128m \n-XX:MaxTenuringThreshold=0 \n-XX:SurvivorRatio=1024 \n-XX:+UseCMSInitiatingOccupancyOnly \n-XX:CMSInitiatingOccupancyFraction=60\n-XX:+DisableExplicitGC",
      "language": "shell"
    }
  ]
}
[/block]
##Advanced JVM Configuration

Below are sets of advanced JVM configurations for applications that might generate high numbers of temporary objects hence triggering long pauses due to garbage collection activities.

For JDK 1.7⁄ 1.8 (8GB heap example for machine with 32 CPUs):
[block:code]
{
  "codes": [
    {
      "code": "-server \n-Xms8g \n-Xmx8g \n-XX:+UseParNewGC \n-XX:+UseConcMarkSweepGC \n-XX:+UseTLAB \n-XX:NewSize=128m \n-XX:MaxNewSize=128m \n-XX:MaxTenuringThreshold=0 \n-XX:SurvivorRatio=1024 \n-XX:+UseCMSInitiatingOccupancyOnly \n-XX:CMSInitiatingOccupancyFraction=40\n-XX:MaxGCPauseMillis=1000 \n-XX:InitiatingHeapOccupancyPercent=50 \n-XX:+UseCompressedOops\n-XX:ParallelGCThreads=8 \n-XX:ConcGCThreads=8 \n-XX:+DisableExplicitGC",
      "language": "shell"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "body": "Please note that these settings might not always be ideal so always make sure to rigorously test prior to production deployment"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Recommendations for clusters with OFF_HEAP caches"
}
[/block]
This section contains basic recommendations for clusters that keeps all or significant amount of cache entries off heap. 

##JVM configuration

Below are sets of advanced JVM configurations for applications that might generate high numbers of temporary objects hence triggering long pauses due to garbage collection activities.

For JDK 1.8 we recommend to use G1 garbage collector and below you can see 10GB heap example for machine with 64 CPUs with G1 being turned on:
[block:code]
{
  "codes": [
    {
      "code": "-server \n-Xms10g \n-Xmx10g\n-XX:NewSize=512m\n-XX:SurvivorRatio=6\n-XX:+AlwaysPreTouch\n-XX:+UseG1GC\n-XX:MaxGCPauseMillis=2000\n-XX:GCTimeRatio=4\n-XX:InitiatingHeapOccupancyPercent=30\n-XX:G1HeapRegionSize=8M\n-XX:ConcGCThreads=16\n-XX:G1HeapWastePercent=10\n-XX:+UseTLAB\n-XX:+ScavengeBeforeFullGC\n-XX:+DisableExplicitGC\n",
      "language": "shell"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "body": "Use the latest versions of Oracle JDK 8 or Open JDK 8 if you decide to use G1 collector since it has been being constantly improved."
}
[/block]
If by some reason G1 doesn't suite well for your case or JDK 7 is used then you can refer to the following CMS based settings as a good point to start with JVM tuning (10GB heap example for machine with 64 CPUs):
[block:code]
{
  "codes": [
    {
      "code": "-server\n-Xms10g\n-Xmx10g\n-XX:NewSize=512m\n-XX:SurvivorRatio=6\n-XX:+UseParNewGC\n-XX:+UseConcMarkSweepGC\n-XX:+AlwaysPreTouch\n-XX:CMSInitiatingOccupancyFraction=30\n-XX:+UseCMSInitiatingOccupancyOnly\n-XX:+CMSClassUnloadingEnabled\n-XX:+CMSPermGenSweepingEnabled \n-XX:ConcGCThreads=16\n-XX:+UseTLAB\n-XX:+ScavengeBeforeFullGC\n-XX:+CMSScavengeBeforeRemark\n-XX:+DisableExplicitGC\n",
      "language": "shell"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "danger",
  "body": "Please note that these settings might not always be ideal so always make sure to rigorously test prior to production deployment"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "GC attacks by Linux"
}
[/block]
In Linux environment it may happen that an IMDG based application faces with long GC pauses due to I/O or memory starvation or other kernel specific settings. This section gives some guidelines on how to modify kernel settings in order to overcome long GC pauses caused by the Linux kernel.
[block:callout]
{
  "type": "danger",
  "title": "",
  "body": "All the shell scripts commands given below were tested under RedHat 7. They may differ for your Linux distribution.\n\nAlso be sure to check with system statistics, logs that a problem really valid for your case before applying any kernel based settings.\n\nFinally it's advisable to consult with your IT department before making changes at the Linux kernel level in production."
}
[/block]
##I/O issues

If GC log shows “low user time, low system time, long GC pause” then a reason could be with GC threads stuck in kernel waiting for I/O. Basically it happens due to journal commits or file system flush of changes by gzip of log rolling.

As a solution you can increase pages flushing to disk from defaul 30 seconds to 5 seconds
[block:code]
{
  "codes": [
    {
      "code": "\n  sysctl –w vm.dirty_writeback_centisecs = 500\n  sysctl –w vm.dirty_expire_centisecs = 500\n",
      "language": "shell"
    }
  ]
}
[/block]
##Memory issues

If GC log shows “low user time, high system time, long GC pause” then most likely memory pressure triggers swapping or scanning for free memory.

- Check and decrease 'swappiness' setting to protect heap and anonymous memory
[block:code]
{
  "codes": [
    {
      "code": "sysctl –w vm.swappiness=10",
      "language": "shell"
    }
  ]
}
[/block]
- Add –XX:+AlwaysPreTouch to JVM settings on startup
- Turn off NUMA zone-reclaim optimization
[block:code]
{
  "codes": [
    {
      "code": "sysctl –w vm.zone_reclaim_mode=0",
      "language": "shell"
    }
  ]
}
[/block]
- Turn off Transparent Huge Pages if RedHat distribution is used
[block:code]
{
  "codes": [
    {
      "code": "echo never > /sys/kernel/mm/redhat_transparent_hugepage/enabled\necho never > /sys/kernel/mm/redhat_transparent_hugepage/defrag\n",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Getting Heap Dump on Out of Memory Errors"
}
[/block]
In case your JVM is throwing an ‘OutOfMemoryException’ and the JVM process should be restarted you may add the following properties to your JVM configuration:
[block:code]
{
  "codes": [
    {
      "code": "-XX:+HeapDumpOnOutOfMemoryError \n-XX:HeapDumpPath=/path/to/heapdump\n-XX:OnOutOfMemoryError=“kill -9 %p” JROCKIT \n-XXexitOnOutOfMemory",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Detailed Garbage Collection stats"
}
[/block]
In order to capture detailed information about garbage collection and its performance add the following parameters to the JVM configuration:
[block:code]
{
  "codes": [
    {
      "code": "-XX:+PrintGCDetails\n-XX:+PrintGCTimeStamps\n-XX:+PrintGCDateStamps\n-XX:+UseGCLogFileRotation\n-XX:NumberOfGCLogFiles=10\n-XX:GCLogFileSize=100M\n-Xloggc:/path/to/gc/logs/log.txt\n",
      "language": "shell"
    }
  ]
}
[/block]
For G1 it's recommended to set the property below that provides many ergonomic details that are purposefully kept out of the -XX:+PrintGCDetails
[block:code]
{
  "codes": [
    {
      "code": "-XX:+PrintAdaptiveSizePolicy",
      "language": "shell"
    }
  ]
}
[/block]
Make sure you modify the path and file names accordingly and ensure to use a different file name for each invocation in order to avoid overwriting the log files from multiple processes.
[block:api-header]
{
  "type": "basic",
  "title": "FlightRecorder Settings"
}
[/block]
In cases when you need to debug performance or memory issues you can rely on Java Flight Recorder tool that allows continuously collect low level and detailed runtime information enabling after-the-fact incident analysis. To enable Flight Recorder use the following settings below:
[block:code]
{
  "codes": [
    {
      "code": "-XX:+UnlockCommercialFeatures\n-XX:+FlightRecorder\n-XX:+UnlockDiagnosticVMOptions\n-XX:+DebugNonSafepoints",
      "language": "shell"
    }
  ]
}
[/block]
To start recording for a particular Java process use this command as an example
[block:code]
{
  "codes": [
    {
      "code": "jcmd <PID> JFR.start name=<recordcing_name> duration=60s filename=/var/recording/recording.jfr settings=profile",
      "language": "shell"
    }
  ]
}
[/block]
For complete details on Java Flight Recorder refer to Oracle official documentation.