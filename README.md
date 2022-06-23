# ROSMonitoring
*Note: This version is the development version for ROSMonitoring and ROS2. It has been tested on ROS Galactic with Ubuntu 20.04*
*This README is a work-in-progress but contains the basic instructions*

ROSMonitoring is a framework developed for verifying at runtime the messages exchanged in a ROS system.
The repository contains the Python implementation for integrating RML (Runtime Monitoring Language, https://rmlatdibris.github.io/) verification and ROS (https://www.ros.org/). Through instrumentation, it is generated a node monitor in ROS which is able to percept the messages exchanged by the other nodes. In the online application, upon each message reception, the monitor will send a corresponding Json message to a oracle, which has been implemented in this case through a Webserver Prolog attached to an RML specification. In the offline application, the monitor will simply generate a log file which can be easily analyzed later on (in this case always through Prolog and RML). ROSMonitoring is easily extendable anyway to new formalisms; it requires only an oracle Webserver using Websockets ready to receive Json messages generated by the ROS node monitor. In the current implementation, the Webserver is written in SWI-Prolog and, upon a message reception, queries an RML specification.

# Docker

A Docker image containing ROSMonitoring is available with Ubuntu 18.04 and ROS Melodic.   
A tutorial on how to use the Docker version of ROSMonitoring is available here: https://github.com/autonomy-and-verification-uol/ROSMonitoring/wiki/Running-ROSMonitoring-in-Docker   
Otherwise, below we explain how to install and use ROSMonitoring normally (without Docker).


# Prerequisities

ROSMonitoring works for all ROS distributions starting with (including) Groovy Galapagos.
It has been tested up to ROS Noetic, with Ubuntu 20.04.    

## Pip (https://pypi.org/project/pip/)
On Ubuntu 18.04 would be:
```bash
$ sudo apt install pip
```
For other distributions, or if this command does not work, follow the instructions at the link reported above.

Using pip we can then install the Python libraries we need.
```bash
$ pip install websocket_client
$ pip install rospy_message_converter
$ pip install pyyaml
```

## Prolog (http://www.swi-prolog.org/build/PPA.html):
In order to use RML oracle we need to install SWI-Prolog. If the user is not interested in using the RML Oracle, this step can be skipped.
```bash
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:swi-prolog/stable
$ sudo apt-get update
$ sudo apt-get install swi-prolog
```

## Reelay (https://github.com/doganulus/reelay)
In order to use TL oracle we need to install Reelay. If the user is not interested in using the TL Oracle, this step can be skipped.
To install Reelay, follow the instructions at: https://github.com/doganulus/reelay/blob/master/docs/install.md

Note: Reelay requires Python 3.

Using pip:
```bash
$ python -m pip install reelay
```

## Java (https://openjdk.java.net/install/):
The following instructions are for installing OpenJDK-11.
```bash
$ sudo add-apt-repository ppa:openjdk-r/ppa
$ sudo apt-get update
$ sudo apt-get install openjdk-11-jdk
```

# How ROSMonitoring is organized

This repository contains two folders:
 - generator
 - oracle
 - monitor

# Generator

The generator folder contains the generator program (Python). It can be used for instrumenting a ROS project (where the nodes are implemented in Python) and generating a monitor node for achieving the Runtime Verification of our ROS nodes.
This generator program takes a configuration file in input (the config.yaml contained in the same folder). Using this simple configuration file we can customize the generation of our monitors and how the ROS nodes will be instrumented.
*Note: The ROS 2 compatible generator is the folder called ros2_devel*

# Oracle

The oracle folder contains two subfolders:
 - RMLOracle, an implementation exploiting the RML formalism (https://rmlatdibris.github.io/)
 - TLOracle, an implementation exploiting the Reelay Python library (https://github.com/doganulus/reelay).

Since the oracle is completely decoupled from the ROS monitor, other implementations can be easily integrated.  


## RML Oracle

It contains two subfolders: prolog and rml.

The Prolog folder contains the prolog files implementing the semantics of the specification language chosen: RML.
In this folder we can find the semantics of the Trace Expression formalism (the lower level calculus obtained compiling RML specifications). Beside the semantics, we have the implementation of a monitor in Prolog, both for Online and Offline RV. The Online RV is achieved through the use of Websockets; the monitor in Prolog consists in a Webserver listening on a chosen url and port. The ROS monitor generated through instrumentation will communicate the observed events at Runtime through this websocket connection. The Offline implementation is simpler, it simply consists in a Prolog implementation where a log file can be analysed offline (after the execution of the ROS system). Also in this case, the events checked by the monitor are obtained by the ROS monitor, which in the Offline scenario logs the observed events inside a log file. The same log file will be later analysed by the prolog monitor.

The other folder contains example of specifications using RML.

## TL Oracle

It contains three Python files.
 - oracle.py - where we can find the implementation of the Python oracle using the Reelay library. This oracle creates a Webserver (using Websockets) listening on a chosen port (passed as argument to the script). Each event observed on this connection is going to be analysed using Reelay.
 - property.py - where the user can define the property to be checked by the oracle. In this file the user can decide the kind of temporal property to use (Past-LTL, Past-MTL, or Past-STL), and how to convert the messages to the corresponding predicates used inside the property.
 - websocket_server.py - implementation of Websocket to create Python Webserver.

# How to use ROSMonitoring (through an example extracted by ROS Tutorial)

First things first..
Before going on we need a machine with ROS installed. It is not important which ROS distribution, as long as rlcpy is supported.

In the following we are going to use ROS 2 Galactic with Colcon on Ubuntu 20.04, but as mentioned before, you can use any distribution.

## Install ROS 2 Galactic

https://docs.ros.org/en/galactic/Installation.html

## Create a workspace 

https://docs.ros.org/en/galactic/Tutorials/Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace.html

## Create ROS package

https://docs.ros.org/en/galactic/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html



We need the 'py_pubsub' package, so do not forget to create it!

## Writing simple Publisher and Subscriber using rospy

https://docs.ros.org/en/galactic/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html

At the end of this tutorial you should have the talker and listener node working.

At the end of the tutorial, the talker and listener nodes should be able to communicate freely.

In order to simplify the monitoring process and make it easier, we need to change a small thing inside publisher_member_function.py

(We're chaning the _topic_ name from 'topic' to 'chatter')
Line 11 becomes: 
```python 
        self.publisher_ = self.create_publisher(String, 'chatter', 10)
```


(We're simplifying the output)
Line 18 must become:
```python
...
msg.data = 'hello'
...
```

We also need to change the topic name in subscriber_member_function.py
Line 13 becomes: 
```python 
        self.subscription = self.create_subscription(
            String,
            'chatter',
            self.listener_callback,
            10)
```

The last thing to do is to add a launch file for running our nodes.
Create a launch file called 'run.launch' inside the 'py_pubsub' folder, and paste the following XML inside it.
```xml
<launch>
    <node pkg="py_pubsub" exec="talker" name="talker" output="screen"/>
    <node pkg="py_pubsub" exec="listener" name="listener" output="screen"/>
</launch>
```
Now we are ready to start monitoring our talker and listener nodes!

## Clone the ROSMonitoring repository

We need the ROSMonitoring implementation in order to instrument and verify our nodes. So, now is the time to clone the repository, if you have not already.

In the terminal:
```bash
 $ cd ~/
 $ git clone https://github.com/autonomy-and-verification-uol/ROSMonitoring.git
 $ git checkout ros2
```
Now you should have your local ROSMonitoring folder.
*Note: Please checkout this branch for ROS 2*

### Create a simple Offline monitor

The creation of a monitor is extremely flexible, and we can easily customize how many monitors, what they can do, and above all, what they are going to check (which topics, and so on).
For customizing the monitors, we use a YAML configuration file. You can find different ones we already prepared for you for exploring ROSMonitoring in the Talker-Listener example.

The first we are going to see is: 'offline_config.yaml'

```yaml
path: /home/parallels/dev_ws/src # this is the path to the ros workspace you'd like the monitor package in
nodes: # here we list the nodes we are going to monitor
  - node:
      name: talker
      package: py_pubsub
      path: /home/parallels/dev_ws/src/py_pubsub/run.launch
  - node:
      name: listener
      package: beginner_tutorials
      path: /home/parallels/dev_ws/src/py_pubsub/run.launch

monitors: # here we list the monitors we are going to generate
  - monitor:
      id: monitor_0
      log: ./log.txt # file where the monitor will log the observed events
      silent: False # we let the monitor to print info during its execution
      topics: # the list of topics this monitor is going to intercept (only one here)
        - name: chatter # name of the topic
          type: std_msgs.msg.String # type of the topic
          action: log # the monitor will log the messages exchanged on this topic
```

This configuration file informs the generator about two nodes: talker and listener. Along with important information concerning their package and where we can find the corresponding launch file (which is the one we created previously).  

Now we can run the generator passing this configuration file in the following way.

```bash
$ cd ~/ROSMonitoring/generator/ros2_devel/
$ chmod +x generator
$ ./generator --config_file offline_config.yaml
```

Going back to the 'dev_ws' folder, if we look into the 'src/monitor/monitor/' folder, we will find a new generated Python script called 'monitor_0.py'. This file contains the code for the monitor.
Inside 'py_pubsub' we can also find now a new launch file called 'run_instrumented.launch'.

Now, if we want to run our ROS nodes with the new monitor together. Since we are adding a new ROS package (the monitor package), we need also to re-run the colcon build command. Before we do this, we need to delete the build, install and log folders because we have just copied a C++ package and that might conflict with paths.

Now we have everything we need to run the system along with the monitor.

In a terminal we do:

```bash
$ cd ~/dev_ws/
$ ros2 launch src/monitor/launch/monitor.launch
```

Then, in another terminal we do:

```bash
$ cd ~/dev_ws/
$ ros2 launch src/py_pubsub/run_instrumented.launch
```

You should not notice any difference, even though now we have a monitor running along with the two other nodes.
What is it actually happening? We have created and run an offline monitor. If we stop the nodes and the monitor, we should see that a new file has been created inside 'dev_ws', called 'log.txt' (as we set in the config file).

We can find the automatically generated log file (log.txt) inside ~/catkin_ws folder.

The log file should look like this:
```json
{"topic": "chatter", "data": "hello", "time": 1559638159.43485}
{"topic": "chatter", "data": "hello", "time": 1559638159.534461}
{"topic": "chatter", "data": "hello", "time": 1559638159.635648}
...
```

The so generated log file can be parsed by any runtime monitor, as long as the latter supports events formatted using Json.
The default Oracle for ROSMonitoring is implemented in SWI-Prolog and supports the RML formalism.

The last step for the Offline version is to check the log file against a formal specification.
To do this, first we copy the log file into the prolog folder, and then we run the monitor (using the already given sh file).
```bash
$ cp ~/catkin_ws/log.txt ~/ROSMonitoring/oracle/
$ cd ~/ROSMonitoring/oracle/RMLOracle/prolog/
$ sh offline_monitor.sh ../rml/test.pl ../../log.txt
...
matched event #89
matched event #90
matched event #91
matched event #92
Execution terminated correctly
```

offline_monitor.sh expects two arguments:
 - the specification we want to verify (test.pl in this example)
 - the log file containing the traces generated by the ROS monitor (log.txt in this case)

The test.pl is the lower level representation of test.rml (contained in the same folder). If we want to verify new properties, we only need to write them followin the RML syntax (creating a corresponding .rml file). And then, we can compile the new rml specifications using the rml-compiler.jar (also contained in the rml folder).

For instance, to generate test.pl, we can do as follows:
```bash
$ cd ~/ROSMonitoring/oracle/RMLOracle/rml/
$ java -jar rml-compiler.jar --input test.rml --output test.pl
```
The compiler will automatically compile the rml file into the equivalent prolog one, which can be used directly from the Prolog monitor.
More information about RML can be found at: https://rmlatdibris.github.io/

Alternatively, we can analyse the log file using the TL oracle.
```bash
$ cd ~/ROSMonitoring/oracle/TLOracle/
$ ./oracle.py --offline --property property --trace ../../log.txt --discrete
```
The TL property defined now into 'property.py' is only an example (in the chatter example there is nothing interesting enough to be checked). The property says that 'chatter' is always contained inside the message.
Note:

--discrete means that we are assuming the events homogeneously distributed (the time between two events is fixed)

--dense means that the events can be observed at a different rate (the time between two events is not always the same)

### Adding a monitor in the middle.

In the previous example we saw how to generate a monitor which logs the intercepted events. In that scenario we can achieve in this way offline RV, because we are analyzing previously generated traces. But, with ROSMonitoring we can do much more than that. We can create a monitor which achieves online RV, meaning that the analysis is done while the system is running.

Let's have a look at the other configuration file called: 'online_config.yaml'

```yaml


path: /home/parallels/dev_ws/src/ # this is the path to the ros workspace you'd like the monitor package in
nodes: # here we list the nodes we are going to monitor
  - node:
      name: talker
      package: py_pubsub
      path: /home/parallels/dev_ws/src/py_pubsub/run.launch
  - node:
      name: listener
      package: py_pubsub
      path: /home/parallels/dev_ws/src/py_pubsub/run.launch

monitors: # here we list the monitors we are going to generate
  - monitor:
      id: monitor_0
      log: ./log.txt # file where the monitor will log the observed events
      silent: False # we let the monitor to print info during its execution
      oracle: # the oracle running and ready to check the specification (localhost in this case)
        port: 8080 # the port where it is listening
        url: 127.0.0.1 # the url where it is listening
        action: nothing # the oracle will not change the message
      topics: # the list of topics this monitor is going to intercept
        - name: chatter # name of the topic
          type: std_msgs.msg.String # type of the topic
          action: filter
          publishers:
           - talker
```

This configuration file is very similar to the previous one. But this time we are asking for the generation of an online monitor. In order to do so, we need to inform the generator where the Oracle is listening and on which port. In this way, the generated monitor will be capable of communicating with it using WebSockets.
Another addition to this configuration file is the 'publishers' field inside the chatter topic.
Since we are doing online RV, the monitor is checking the events at runtime. Now, if we wanted just to log each event, we could maintain the action set to 'log'. The behaviour in this way would be exactly the same as for the offline monitor, with the only difference that each time an event is observed, the monitor propagates this event to the oracle and waits for the current verdict against a chosen property. Consequently, rather than the offline case, in the online scenario, the monitor will also log the satisfaction/violation of the property (but nothing more). This can be useful if we are debugging a system, but in a real scenario we could need to enforce the correctness of the events. For instance, filtering the events which are considered wrong by the Oracle. For doing this, we can change the action from 'log' to 'filter'.

Once the action 'filter' is selected, the monitor will filter the wrong messages. But, to be able to do so, it must be in the middle of the communication. Until now the monitor was only another node in the system and was just subscribing the topics. This is not enough if we want to filter the wrong messages. In order to solve this problem, ROSMonitoring instrument the nodes changing the names and creating gaps in the communications. Thanks to this communication gaps, the monitor can become a bridge for the topics of our interest, and filter the messages in case they are wrong.

To create the gap the generator needs to know who is the publisher (or subscriber) for the topic we want to filter. In this case we indicate 'talker', which is the publisher for the 'chatter' topic.

After that, we can simply run again the generator.

```bash
$ cd ~/ROSMonitoring/generator/ros2_devel/
$ chmod +x generator
$ ./generator --config_file online_config.yaml
```

This will generate again a new monitor and the launch files we need.
As before, we run colcon build again.

Since now the monitor is online, it needs an oracle to check the events. As for the offline case, ROSMonitoring does not require any specific runtime monitor to be used as Oracle. The only requirements are having an Oracle capable of communicating through WebSockets and able to parse Json events.
Again, ROSMonitoring already have a default Oracle, which is implemented in SWI-Prolog and supports the RML formalism.

Thus, before running our Online monitor, we need to execute the Webserver Prolog, as possible implementation of our oracle.
```bash
$ cd ~/ROSMonitoring/oracle/RMLOracle/prolog/
$ sh online_monitor.sh ../rml/test.pl 8080
% Started server at http://127.0.0.1:8080/
Welcome to SWI-Prolog (threaded, 64 bits, version 8.0.2)
SWI-Prolog comes with ABSOLUTELY NO WARRANTY. This is free software.
Please run ?- license. for legal details.

For online help and background, visit http://www.swi-prolog.org
For built-in help, use ?- help(Topic). or ?- apropos(Word).

?-
```

After that we can do the same as for the offline case. First we run 'run.launch' for running the monitor, and then we run 'run_instrumented.launch' for running the instrumented nodes (notice that now we added the 'remap' params fro creating the gap in the communication).
The monitor will now check the events at runtime. But, since the property is always satisfied, no events will be filtered out. Changing the property you will be able to see that if the event is not consistent, it is not propagated to the subscriber node!

Alternatively, we can use the online oracle with TL properties.
```bash
$ cd ~/ROSMonitoring/oracle/TLOracle/
$ ./oracle.py --online --property property --port 8080 --discrete

```
In this way, the Python oracle will start listening on the 8080 port. Each message observed on this connection will be passed to Reelay and checked against the property defined in property.py.

## Additional information published by the monitor (online)

The monitor reports constantly information about its analysis.

- When a violation of the formal property is observed, the monitor publishes a message on the topic '/{monitor_id}/monitor_error' of type MonitorError (monitor/msg/MonitorError.msg). Where {monitor_id} is the monitor id added in the configuration file (see generation section above). For instance, if monitor id is 'monitor_1', then the error messages will be reported on the topic '/monitor_1/monitor_error'.
- On the topic '/{monitor_id}/monitor_verdict' it is possible to keep track of the current monitor's verdict. The message is of type String, and can be: 'true', 'false', 'currently_true', 'currently_false', 'unknown'. Depending on the chosen oracle, all or a subset of these verdicts can be reported by the monitor.  

Note: To enable the publishing of error messages, the user has to set the warning field to 1 or 2 (0 no warnings will be published). If the warning level is set to 1, then an error will be reported when the monitor's verdict is 'currently_false' or 'false'. If the warning level is set to 2, then an error will be reported only when the monitor's verdict is 'false'. Thanks to the warning level we can customise how strict we want to be on the monitor's verdict. There might be scenarios where we might be more interested in checking a property against the system (in this case warning level 2, since we only care about the system satisfying/violating the property); while there might be other scenarios where we care about the current satisfaction/violation of a property in the current system state (warning level 2), even though in the future the verdict might change. In practice, warning level has to be set to 2 when we only care about final verdicts, so we want the monitor to report an error only when it has proven the property has been violated by the system and will always be violated; warning level to 1, when we are not interested only in the final verdict, and we want to give importance to the current observed trace, and its satisfaction/violation of the property under analysis. In this case, the current state of the system might be satisfying/violating the property, but the monitor cannot conclude it will always be satisfied/violated.

# License:
ROSMonitoring project is released under MIT license

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Third-party libraries licenses:
 - websocket_client (https://pypi.org/project/websocket_client/)

    Copyright 2018 Hiroki Ohtani (BSD License)
 - rospy_message_converter (https://github.com/uos/rospy_message_converter)

    Copyright (c) 2013, Willow Garage, Inc. (BSD License)
 - python-websocket-server (https://github.com/Pithikos/python-websocket-server)

    Copyright (c) 2018 Johan Hanssen Seferidis (MIT License)
 - reelay (https://github.com/doganulus/reelay)

    Copyright (c) 2019 Dogan Ulus (Mozilla Public License Version 2.0)
 - RML (https://github.com/RMLatDIBRIS)

    Copyright (c) 2019 RMLatDIBRIS (Mozilla Public License Version 2.0)
