Starting in Junos OS Release 18.1R1, a new sensor is available that allows syslog data to be streamed to network telemetry collector systems.  
Using the `/junos/events/` sensor, you can now stream syslog messages to your telemetry-collection systems.  
So the same gRPC telemetry collector can be used to subscribe to syslog messages and to openconfig paths.  

# About this repo

We will subscribe to Junos syslog events to stream them to a gRPC telemetry collector  
We will use jtimon. 

# About Jtimon

jtimon is a grpc client.  
It is opensourced and written in GO.  
https://github.com/nileshsimaria/jtimon  
jtimon can also export the data received from Junos devices to Influxdb, Prometheus, ... 

# Demo

## About the lab

we will use one Junos device and one ubuntu VM.  

## requirements

### Junos requirements

This feature is available from Junos 18.1R1. 

Here's my device details: 

```
jcluser@vMX-addr-0> show version | match "Junos:|openconfig|na telemetry"
Junos: 18.2R1.9
JUNOS na telemetry [18.2R1-S3.2-C1]
JUNOS Openconfig [0.0.0.10-1]
```
```
jcluser@vMX-addr-0> show configuration system services extension-service
request-response {
    grpc {
        clear-text {
            port 32768;
        }
        skip-authentication;
    }
}
notification {
    allow-clients {
        address 0.0.0.0/0;
    }
}

```
### Install Docker on the ubuntu VM. 

```
$ sudo apt-get update
```
```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
```
$ sudo apt-get update
```
```
$ sudo apt-get install docker-ce
```
```
$ sudo docker run hello-world
```
```
$ sudo groupadd docker
```
```
$ sudo usermod -aG docker $USER
```

Exit the ssh session and open an new ssh session and run these commands to verify you installed Docker properly:  
```
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```
```
$ docker --version
Docker version 18.03.1-ce, build 9ee9f40
```

## jtimon 

### Build a jtimon Docker image
```
$ git clone https://github.com/nileshsimaria/jtimon.git
$ cd jtimon/
$ make docker
```
### check the image
```
$ docker images jtimon
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jtimon              latest              2e8967d4ea00        2 hours ago         16.4 MB
```
### List running containers

There is no container running
```
$ docker ps | grep jtimon
```

### create a jtimon configuration file

use of the files at the root of this repository

[vmx0.json](vmx0.json) is used to subscribe to all syslog messages   
[vmx0_filter_daemon.json](vmx0_filter_daemon.json) is used to subscribe to all syslog messages sourced from a specific daemon  
[vmx0_filter_event_id.json](vmx0_filter_event_id.json) is used to subscribe to a specific syslog event  
[vmx0_filter_starts_with_event_id.json](vmx0_filter_starts_with_event_id.json) is used to subscribe to syslog events that begin with a certain string  

```
vi vmx0.json
```
### run jtimon 

Lets run jtimon dockerized with the configuration file. Let's print telemetry data.  
```
./jtimon --config vmx0.json --print
```

## Verify on Junos 

### Display information about sensors 
To display information about sensors, run this command on a Junos device:
```
jcluser@vMX-addr-0> show agent sensors 
```
###  verify if there is an established connection between jtimon and a Junos device 
To verify if there is an established connection between jtimon (grpc client) and a Junos device (grpc server), run this command on a Junos device:
```
jcluser@vMX-addr-0> show system connections | grep 32768
tcp4       0      0  100.123.1.0.32768                             100.123.35.0.50808                            ESTABLISHED
tcp46      0      0  *.32768                                       *.*                                           LISTEN
```

## generate a custom syslog message from the Junos device

To generate a custom syslog message from the Junos device, run this command: 
```
jcluser@vMX-addr-0> start shell
% logger -e EVENT_FAKE -d mgd "THIS IS A FAKE SYSLOG EVENT"
% exit
```
## jtimon output
```
system_id: vMX-addr-0
component_id: 65535
sub_component_id: 0
path: sensor_1000:/junos/events/:/junos/events/:eventd
sequence_number: 11
timestamp: 1552828143258
sync_response: false
  key: __timestamp__
  uint_value: 1552828143259
  key: __junos_re_stream_creation_timestamp__
  uint_value: 1552828143258
  key: __junos_re_payload_get_timestamp__
  uint_value: 1552828143258
  key: __junos_re_event_timestamp__
  uint_value: 1552828143258
  key: __prefix__
  str_value: /junos/events/event[id='EVENT_FAKE' and type='2' and facility='1']/
  key: timestamp/seconds
  uint_value: 1552828143
  key: timestamp/microseconds
  uint_value: 257801
  key: priority
  uint_value: 5
  key: pid
  uint_value: 71830
  key: message
  str_value: THIS IS A FAKE SYSLOG EVENT
  key: daemon
  str_value: mgd
  key: hostname
  str_value: vMX-addr-0
  key: logoptions
  int_value: 0

```

# Looking for others jtimon demos

https://github.com/ksator/collect_telemetry_from_junos_with_jtimon  
https://github.com/ksator/junos_monitoring_with_prometheus  

# Credits

Thank you to Jag Channa of OpenEye Software for writing this blog: https://openeye.blog/2018/04/05/streaming-syslog-events-through-junos-telemetry-interface/. It provided the basis for this repository. 

