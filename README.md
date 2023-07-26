# Local Kafka


This repo consists of:

- Docker file to bundle confluent kafka image with a jmx agent
- Docker compose creating:
  - 3 node kafka cluster
  - 3 zookeeper nodes
  - Prometheus
  - Grafana


### Kafka + JMX Exporter

The Docker file bundles together the latest (just for testing, recommend pinning a version for more vigour)
Confluent Kafka image with the Prometheus JMX exporter (at time of creation - July 2023 - latest version).
As part of this a .yaml config file is also included for the JMX exporter picking up a range of Kafka metrics.

To build the image run:
```
docker build -t kafka-with-jmx-exporter .
```
#### Using the Kafka with JMX Exporter image

The bundling of the JMX exporter and Kafka image together means that the Exporter and Kafka are running on the same JVM.
The Exporter thus will not need a network interface to access the JMX data.
In the Docker Compose file the parameters to use the Exporter are passed via the `KAFKA_OPTS` environment variable.
This variable is used to pass options to the JVM running Kafka. In this case, it's specifying to use the Prometheus JMX Exporter 
Java Agent that was bundled into the image. The agent needs to know the port on 
which to expose the metrics and where the configuration file is located (`/usr/app/jmx-exporter-config.yml`).

Why the other Kafka environment variables in regard to JMX (`KAFKA_JMX_OPTS`,`KAFKA_JMX_HOSTNAME`, `KAFKA_JMX_PORT`)?
These are not strictly needed as we have decided to use the JMX Exporter to expose metrics to Prometheus. These have been
included and exposed on a different port to allow the user to examine the JMX via tooling such as `jconsole` to explore
the MBeans. These settings are in effect the network interface for an external client to connect the Kafka JMX.


### Prometheus
A very simple setup with the configs scraping the JMX Exporter and Kafka JMX every 15 seconds.
To access Prometheus once started navigate to:
```
http://localhost:9090/
```
in a web browser

### Grafana
This can be accessed at:
```
http://localhost:3000/
```
To log in the username and password are the defaults username: `admin`, password: `admin`.
Follow the GUI to add Prometheus as a data source. The important thing here is the URL to connect to
will be: `prometheus:9090` and not `localhost` as it defaults to.


### A note on the Docker network and localhost
I'm including this Note here as it is something that has confused me time and again over the years.
The Docker Compose file will create a network - in this case `kafka-net`. We stipulate in our compose file
that each of our containers are created within this network. This means that all containers can resolve to each
other using their *container* name and port.
When trying to access these elements from outside the network ie. web browser, we must use `localhost`. It is for this
reason that all the ports exposed on the Kafka containers are different - we might want to examine the metrics from our
browser or external client and so they must all map to different ports on localhost and not clash. Prometheus and Grafana
are again accessed via a web browser and thus resolved via `localhost`. However, for Grafana to get the metrics from 
Prometheus the container name is used as they are running in the same network.
In a Docker Compose the `ports` list are those that are exposed to the local machine. `ports` actually also runs `expose`
which means the ports listed are also available on the internal Docker network. 

