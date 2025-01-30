#  auto.offset.reset  earliest and latest offset values  

When a consumer joins a consumer group it will fetch the last committed offset so it will restart to read from 5, 6, 7 if before crashing it committed the latest offset (so 4).
The earliest and latest values for the auto.offset.reset property is used when a consumer starts but there is no committed offset for the assigned partition.
In this case you can chose if you want to re-read all the messages from the beginning (earliest)
or just after the last one (latest)
When auto.offset.reset is set to latest, there are 2 scenarios can happen: first time when the consumer subscribe to topic, it will only receive the message arrive after it subscribed. Other scenario is when the consumer reconnect to the topic(after get crashed or something), consumer will receive the message 5, 6, 7 because the latest commit was 
