
# Intro

Flume is a ingestion tool for high-volume event-based data.

Example: ingestion of log files of web servers into Hadoop.

In a simplified form, flume works like this:

Source ====== Channel ====> Sink

* Source: consumes events delivered to it by an external source, like spool directory, web server, etc.
* Channel: flume stores in channels the events it resieves from a source, like file or memory.
* Sink: removes the event from the channel and puts it into an external repository like log file, HDFS etc.

# Examples

The examples are taken from [here](https://github.com/tomwhite/hadoop-book/tree/master/ch14-flume)

### Example 1 

* Source: spool directory
* Channel: file
* sink: logger

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

### Example 2 

* Source: spool directory
* Channel: file
* sink: logger

```
agent1.sources = source1
agent1.sinks = sink1
agent1.channels = channel1

agent1.sources.source1.channels = channel1
agent1.sinks.sink1.channel = channel1

# Define a spool source
agent1.sources.source1.type = spooldir
agent1.sources.source1.spoolDir = /tmp/spooldir

# Define hdfs sink 
agent1.sinks.sink1.type = hdfs
agent1.sinks.sink1.hdfs.path = /tmp/flume_test
agent1.sinks.sink1.hdfs.filePrefix = events
agent1.sinks.sink1.hdfs.fileSuffix = .log
agent1.sinks.sink1.hdfs.inUsePrefix = _
agent1.sinks.sink1.hdfs.fileType = DataStream

# Define a file channel
agent1.channels.channel1.type = file
```
