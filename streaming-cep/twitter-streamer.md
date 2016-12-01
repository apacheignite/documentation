Ignite Twitter Streamer module consumes tweets from Twitter and feeds the transformed key-value pairs <tweetId, text> into an Ignite cache. 

To stream data from twitter into an Ignite cache, you need to: 

1. Import Ignite Twitter Module in Maven Project
If you are using Maven to manage dependencies of your project, you can add the Twitter module
dependency like this :
[block:code]
{
  "codes": [
    {
      "code": "<dependency>\n  <groupId>org.apache.ignite</groupId>\n  <artifactId>ignite-twitter</artifactId>\n  <version>${ignite.version}</version>\n</dependency>",
      "language": "xml"
    }
  ]
}
[/block]
Replace '${ignite.version}' with the actual Ignite version you are interested in.

2. In your code, set the necessary parameters and start the streamer, like so:
[block:code]
{
  "codes": [
    {
      "code": "IgniteDataStreamer dataStreamer = ignite.dataStreamer(\"myCache\");\ndataStreamer.allowOverwrite(true);\ndataStreamer.autoFlushFrequency(10);\n\nOAuthSettings oAuthSettings = new OAuthSettings(\"setting1\", \"setting2\", \"setting3\", \"setting4\");\n\nTwitterStreamer<Integer, String> streamer = new TwitterStreamer<>(oAuthSettings);\nstreamer.setIgnite(ignite);\nstreamer.setStreamer(dataStreamer);\n\nMap<String, String> params = new HashMap<>();\nparams.put(\"track\", \"apache, twitter\");\nparams.put(\"follow\", \"3004445758\");\n\nstreamer.setApiParams(params);// Twitter Streaming API params.\nstreamer.setEndpointUrl(endpointUrl);// Twitter streaming API endpoint.\nstreamer.setThreadsCount(8);\n\nstreamer.start();",
      "language": "java"
    }
  ]
}
[/block]
Refer to [Twitter streaming API](https://dev.twitter.com/streaming/overview) documentation for more information on various streaming parameters.