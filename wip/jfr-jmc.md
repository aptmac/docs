# Profiling Java Applications JDK Flight Recorder (JFR) and JDK Mission Control (JMC)

JDK Flight Recorder (JFR) is a production time profiling and diagnostics engine built into the JVM.

JDK Mission Control (JMC) is, among other things, the client tool used to look at recordings produced by the JDK Flight Recorder.

## 1. JDK Flight Recorder (JFR)
### 1.1 What is JDK Flight Recorder

JDK Flight Recorder manages data gathered from the JVM. Data is written in memory, first to thread local buffers, and then promoted to a fixed-size global ring buffer. Traditionally, this data is eventually flushed to JFR files (*.jfr) on disk, which is then consumed for analysis, for example by JDK Mission Control.

**Recordings**

Users manage the system’s recordings. Individual recordings have unique configuration and can be started, stopped, and dumped to disk on demand.

**Events**

Events are the basic unit of JFR data. The JVM has pre-existing events and there is API for users to supply both static and dynamic custom events. Every event can be enabled or disabled when recording to minimize overhead.

JFR profiles, or event templates, (*.jfc) can be used to configure the events for a recording. The JDK distribution comes with two profiles: 'default' and 'profile'.

default: Low overhead configuration safe for continuous use in production environments, typically less than 1 % overhead.

profile: Low overhead configuration for profiling, typically around 2 % overhead.

## 2. JDK Mission Control (JMC)
### 2.1 What is JDK Mission Control

JDK Mission Control is an open source production time profiling and diagnostics tool for Java.

JMC has two main ways of presenting information: real-time monitoring of running JVMs, and historical analysis via flight recordings and hprof files (as of JMC 7.1.0).

### 2.2 Real-Time Monitoring

JMC is able to deliver real-time information via the MBean server. This can be observed via the JVM Browser tab inside of JMC, by either right-clicking a running JVM and selecting the "Start JMX Console" option, or by expanding the displayed JVM tree menu and double-clicking the "MBean Server" item.

There are a handful of pages for displaying information pertaining to the JVM.

The Overview page displays information such as CPU and memory usage, in both graph form and in a dashboard with dials like a pressure gauge.

There are dedicated pages for viewing specific information such as MBean objects, System information, Memory, and Threads.

There are also two interactive pages: the Triggers and Diagnostic Commands pages.

The Triggers page allows the user to set an action to be fired in response to specific activity in the JVM. These actions could be as simple as displaying an alert dialog or writing a message to the console, or more complicated like starting or dumping a flight recording. Some examples of activity that could trigger an action would be CPU usage surpassing a set value, or threads entering deadlock.

The Diagnostic Commands page allows for the usage of jcmd commands via a GUI. This page displays a list of commands and their options, and upon executing a command, displays the output.  

### 2.3 Historical Analysis (JFR & hprof)

JMC has a separate suite of pages and tools for displaying flight recording information. Flight recording files can be opened via the File menu, or automatically when a flight recording is dumped using JMC. These pages and their functionality will be covered in more detail later, in part 4: Analyzing Flight Recordings using JDK Mission Control.

As of JMC 7.1.0, the JOverflow page has been refactored and included by default with JMC. This allows for the visualization of heap dump hprof files (*.hprof). For a detailed look at how to use the JOverflow page, see a blog post written by the Marcus Hirt (JMC Project Lead): http://hirt.se/blog/?p=854

## 3. Using JDK Flight Recorder

### 3.1 Using JDK Flight Recorder with OpenJDK

#### 3.1.1 Starting JDK Flight Recorder at JVM start time

When starting a Java process, JFR can be started by using the JVM flag: `--XX:StartFlightRecording`.

There are also a couple of ways to adjust the behaviour of JFR by adding extra parameters. This can be performed by adding options to the end of the StartFlightRecording JVM flag `--XX:StartFlightRecording=<options>`, or by using the dedicated flag `--XX:FlightRecorderOptions=<options>`. Options are a comma delimited list of key-value pairs.

Example 1: The following command will start a flight recording at the same time as running the Demo application

`java --XX:StartFlightRecording Demo`

Example 2: The following command will start the Demo application and will initiate a 1 hour long flight recording, which will later be saved to a file called "demorecording.jfr"

`java --XX:StartFlightRecording=duration=1h,filename=demorecording.jfr Demo`

For a more comprehensive and detailed list of JFR options, see the Oracle java documentation: https://docs.oracle.com/en/java/javase/11/tools/java.html

#### 3.1.2 Using JDK Flight Recorder on a currently running JVM

Located under $JAVA_HOME/bin, jcmd is a utility that can be used to send diagnostic command requests to a running JVM. jcmd includes commands for interacting with JFR, with the most basic commands being start, dump, and stop.

In order to interact with a JVM, jcmd requires it's process id. The pid can be retrieved by using `jcmd -l` which displays a list of the running JVM process ids, as well as other information such as the main class and command-line arguments that were used to launch the processes.

- Start a recording with: `jcmd <pid> JFR.start <options>`

Example: The following command will start a recording named "demorecording", which will keep data from the last four hours, and will be limited to a size of 400 MB.

`jcmd _pid_ JFR.start name=demorecording maxage=4h maxsize=400MB`

- Dump a recording with: `jcmd <pid> JFR.dump <options>`

Example: The following command will dump a flight recording file called "dumpedrecording.jfr" from the previously started recording "demorecording"

`jcmd _pid_ JFR.dump name=demorecording filename=dumpedrecording.jfr`

- Stop a recording with: `jcmd <pid> JFR.stop <options>`

Example: The following command will stop the recording named demorecording, and will write its contents to a file called "demorecording.jfr" at the path provided

`jcmd _pid_ JFR.stop name=demorecording filename=/home/user/recordings/demorecording.jfr`

For more information about `jcmd` usage and additional JFR commands and options, see the Oracle jcmd documentation: https://docs.oracle.com/en/java/javase/11/tools/jcmd.html

### 3.2 Using JDK Flight Recorder with JDK Mission Control

#### 3.2.1 JVM Browser & Flight Recording Wizard

JMC has a Flight Recording Wizard that allows for a streamlined experience of starting and configuring flight recordings. This wizard can be launched by either:

1. Right-clicking a JVM in the JVM Browser view and selecting the "Start Flight Recording ..." option from the context menu, or by expanding the JVM
2. Expanding a JVM tree menu in the JVM Browser, and double-clicking the "Flight Recorder" tree item

The Flight Recording Wizard has a total of three pages.

The first page contains general recording settings for the flight recording. These configurable settings include the name of the recording, where the file will be saved to and what it will be named, whether this is a time fixed or continuous recording, which event template will be used, and finally a description of the recording.

The second page contains event options for the flight recording. These settings include the level of detail to be recorded by various types of events, for example, Garbage Collections, Memory Profiling, and Method Sampling, among others.

The third and final page contains settings for the event details, and allows for turning events on or off, enable the recording of stack traces, and altering the time threshold required to record an event.

Once the settings are adjusted for the flight recording, the wizard will exit and start the flight recording once the "Finish" button has been selected.

#### 3.2.2 Triggers Page

Flight recordings can also be started or dumped in response to a user-defined trigger rule. This can be setup in the Triggers page of a running JVM through the JMX Console.

The left side of the Triggers page displays rules and conditions that, when met, will trigger an action. These actions are listed on the right-side of the Triggers page, under the header "Rule Details". The JFR-related actions are: Dump Flight Recording, Start Continuous Flight Recording, and Start Time Limited Flight Recording.

For example: a trigger rule could be setup to dump a flight recording containing the last 30 seconds of data in response to the JVM CPU usage surpassing 75%.

#### 3.2.3 Interacting with a Flight Recording in progress

Once a flight recording has been started, it will appear under the "Flight Recording" tree menu. There are several ways of interacting with a flight recording in progress by right-clicking a recording and using the context menu options. More specifically, the options are:

Dump...: opens a dialog with options to specify the time interval of interest
Dump whole recording: dumps the entire flight recording
Dump last part: by default this is the last 5 minutes
Edit: opens a dialog with the first and third pages of the flight recording wizard
Stop: stops the flight recording, but keeps it in the JVM Browser for dumping
Close: stops and removes the flight recording from the JVM Browser

When a flight recording has been dumped it will be opened automatically by JMC.

## 4. Analyzing Flight Recordings using JDK Mission Control

JMC contains a suite of pages for displaying information from JFR files (*.jfr). These pages can be divided into several categories.

### 4.1 Automated Analysis Results

The Automated Analysis Results page is opened by default when a flight recording file is loaded.

JMC scans the JFR file looking for patterns in the data and tries to identify performance and functional issues. Once analysis is completed, it displays a list of areas where potential problems have been detected, and provides insight into how these warnings may be resolved.

### 4.2 Java Application

JMC includes a handful of pages to visualize and display information related to the execution of the Java Application. The Java Application page includes a high-level view into various aspects of the programs execution, which are later expanded upon and displayed in more detail in subsequent pages. These pages include the Threads, Memory, Lock Instances, File I/O, Socket I/O, Method Profiling, Exceptions, and Thread Dumps pages.

### 4.3 JVM Internals

JMC includes pages to deliver information about the JVM, starting with the JVM Internals page which displays information such as what version of Java was used, and what JVM arguments and flags were used. Additionally, there are dedicated pages for detailing Garbage Collections, GC Configuration, Compilations, Class Loading, VM Operations, and TLAB Allocations.

### 4.4 Environment

The Environment page and subsequent pages contain information about the host machine, including its Processes, Environment Variables, System Properties, Native Libraries, and information about the actual recording.

### 4.5 Event Browser

The Event Browser contains a list of all the events in the flight recording. The page includes a search bar and tree-view for quick inspection, and a table to display details from the recorded events.

### 4.6 Stack Trace & Flame View

In addition to the various JFR-related pages, JMC contains a pane which displays the Stack Trace view by default, and can be updated to optionally include the Flame View. Both of these views update in response to data selections in the JFR-related pages. For example, selecting an entry in the Memory page table will update the Stack Trace and Flame Views to display the information relevant to the selected class.

New to JMC 7.1.0, the Flame View can be added to the Stack Trace pane via a dialog located under Window -> Show View -> Other..., and selecting enabling "Flame View" under the Mission Control tree menu. The Flame View displays stack traces in a graphical representation, where the stack frames are placed on top of each other. This view contains a search bar for quick highlighting of relevant stack frames, and tooltips that upon hovering a stack frame to display additional information.

### 4.7 HPROF & JOverflow

As of JMC 7.1.0, the JOverflow plugin has been re-written and is now included by default in JMC. The JOverflow plugin analyzes heap dump information from *.hprof files, searching for memory usage anti-patterns including duplicate strings, empty arrays, and unused collections. Heap dumps can be performed in JMC by right-clicking a JVM in the JVM Browser and selecting "Dump Heap" from the context menu. When the hprof file is generated, JOverflow will open it automatically. Because the JOverflow plugin analyzes hprof files instead of jfr files, it does not have the same integration with other JMC pages and may best be thought of as it's own indepedent view.

The JOverflow view divides the page into four quadrants for displaying information. The page can be reset at any point in time via a button located in the top-right corner of the page.

The top left quadrant contains an Object Selection table, which displays a list of the anti-patterns that were flagged by the analysis, and how much memory was used and lost as a result.

The top right quadrant contains the Referrer tree table, which displays aggregated reference chains for the current selection.

The bottom left quadrant contains a pie chart and related table, which displays the currently selected information with the ability to filter the data by selecting a class from the table or the chart itself.

The bottom right quadrant contains another pie chart, this time displaying objects grouped by their closest ancestor referrer. Similar to the previous quadrant, the page data can be filtered by interacting with the table or the chart itself.

For a detailed look at how to use the JOverflow page, see a blog post written by the Marcus Hirt (JMC Project Lead): http://hirt.se/blog/?p=854
