# Set the base image
FROM confluentinc/cp-kafka:latest

USER root

# Set working directory
WORKDIR /usr/app

# Download JMX Exporter agent
ADD https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.19.0/jmx_prometheus_javaagent-0.19.0.jar /usr/app/jmx_prometheus_javaagent.jar

# Copy your JMX Exporter configuration file to the container
COPY jmx-exporter/jmx-exporter-config.yml /usr/app/jmx-exporter-config.yml
RUN chmod 755 /usr/app/jmx_prometheus_javaagent.jar

# Expose the JMX Exporter port
EXPOSE 7071