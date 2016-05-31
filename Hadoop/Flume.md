
# Intro

Flume is a ingestion tool for high-volume event-based data.

Example: ingestion of log files of web servers into Hadoop.

# Examples

```
agent1.sources = source1
agent1.sinks = sink1
agent1.channels = channel1

agent1.sources.source1.channels = channel1
agent1.sinks.sink1.channel = channel1

# Define a spool source
agent1.sources.source1.type = spooldir
agent1.sources.source1.spoolDir = /tmp/spooldir

# Define a logger sink that simply logs all events it receives
agent1.sinks.sink1.type = logger

# Define a file channel
agent1.channels.channel1.type = file
```
