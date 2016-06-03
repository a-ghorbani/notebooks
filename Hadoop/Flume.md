
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
* sink: HDFS

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
a1.channels.channel1.type = file
```
### Store data in partitions

```
agent.sinks.sink1.hdfs.path=/path/to/dir/year=%Y/month=%m/hay=%d
```

### Interceptors

An interceptor can modify or even drop events based on any criteria chosen.

```
a1.sources.r1.interceptors = i1 i2
a1.sources.r1.interceptors.i1.type = host
a1.sources.r1.interceptors.i1.preserveExisting = false
a1.sources.r1.interceptors.i1.hostHeader = hostname
a1.sources.r1.interceptors.i2.type = timestamp
```
In the above example the hostname of the server and timestamp will be added to the events.

### Fan out flow

Fanning out means when the flow from one source to multiple channels.

### Multiplexing

Sending some vents to one channel and others to another channel.

### Tiers

 Tiers are intermediate aggregators.
 Tiers can aggregate events from a group of nodes.
 
 
