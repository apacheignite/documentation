This streamer consumes from a MQTT topic and feeds key-value pairs into an `IgniteDataStreamer` instance, using [Eclipse Paho](https://eclipse.org/paho/) as an MQTT client.

You must provide a stream tuple extractor (either a single-entry or multiple-entries extractor) to process the incoming message and extract the tuple to insert.

This streamer supports:

* Subscribing to a single topic or multiple topics at once.
* Specifying the subscriber's QoS for a single topic or for multiple topics.
* Setting [MqttConnectOptions](https://www.eclipse.org/paho/files/javadoc/org/eclipse/paho/client/mqttv3/MqttConnectOptions.html) to enable features like *last will testament*, *persistent sessions*, etc.
* Specifying the client ID. A random one will be generated and maintained throughout reconnections if the user does not provide one.
* (Re-)Connection retries powered by the [guava-retrying library](https://github.com/rholder/guava-retrying). *Retry wait* and *retry stop* policies can be configured.
* Blocking the start() method until the client is connected for the first time.
[block:api-header]
{
  "type": "basic",
  "title": "Code sample"
}
[/block]
Here's a trivial code sample showing how to use this streamer:
[block:code]
{
  "codes": [
    {
      "code": "// Start Ignite.\nIgnite ignite = Ignition.start();\n\n// Get a data streamer reference.\nIgniteDataStreamer<Integer, String> dataStreamer = grid().dataStreamer(\"mycache\");\n\n// Create an MQTT data streamer  \nMqttStreamer<Integer, String> streamer = new MqttStreamer<>();\nstreamer.setIgnite(ignite);\nstreamer.setStreamer(dataStreamer);\nstreamer.setBrokerUrl(brokerUrl);\nstreamer.setBlockUntilConnected(true);\n\n// Set a single tuple extractor to extract items in the format 'key,value' where key => Int, and value => String\n// (using Guava here).\nstreamer.setSingleTupleExtractor(new StreamSingleTupleExtractor<MqttMessage, Integer, String>() {\n    @Override public Map.Entry<Integer, String> extract(MqttMessage msg) {\n        List<String> s = Splitter.on(\",\").splitToList(new String(msg.getPayload()));\n\n        return new GridMapEntry<>(Integer.parseInt(s.get(0)), s.get(1));\n    }\n});\n\n// Consume from multiple topics at once.\nstreamer.setTopics(Arrays.asList(\"def\", \"ghi\", \"jkl\", \"mno\"));\n\n// Start the MQTT Streamer.\nstreamer.start();",
      "language": "java"
    }
  ]
}
[/block]
Refer to the Javadocs of the `ignite-mqtt` module for more info on the available options.