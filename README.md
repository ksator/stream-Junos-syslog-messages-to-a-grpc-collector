Starting in Junos OS Release 18.1R1, a new sensor is available that allows syslog data to be streamed to network telemetry collector systems.  
Using the /junos/events/ sensor, you can now stream syslog messages event data to your telemetry-collection systems.

## About this repo

We will subscribe to Junos syslog events to stream them to a gRPC telemetry collector
We will use jtimon. 

## Credits

All the credits go to https://techmocha.blog/2018/04/05/streaming-syslog-events-through-junos-telemetry-interface/

## About Jtimon

jtimon is a grpc client. 
It is opensourced and written in GO. 
https://github.com/nileshsimaria/jtimon 

## Looking for others jtimon demos

https://github.com/ksator/collect_telemetry_from_junos_with_jtimon
https://github.com/ksator/junos_monitoring_with_prometheus

## Junos requirements 

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

