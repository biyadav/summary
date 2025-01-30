# kafka earliest and latest offset values  

When a consumer joins a consumer group it will fetch the last committed offset so it will restart to read from 5, 6, 7 if before crashing it committed the latest offset (so 4).
The earliest and latest values for the auto.offset.reset property is used when a consumer starts but there is no committed offset for the assigned partition.
In this case you can chose if you want to re-read all the messages from the beginning (earliest)
or just after the last one (latest)
