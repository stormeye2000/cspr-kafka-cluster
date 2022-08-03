FROM ubuntu:16.04 as downloads

LABEL maintainer="Stormeye 2000"

# Adapted from the Debezium dockerfile
# https://hub.docker.com/r/debezium/kafka/dockerfile/
#
# Used in kubernetes deployments
#
# Set the version, home directory, and MD5 hash.
# MD5 hash taken from http://kafka.apache.org/downloads.html for this version of Kafka
# These argument defaults can be overruled during build time but compatibility cannot be guaranteed when the defaults are not used.
#
ARG KAFKA_VERSION=3.2.1
ARG SCALA_VERSION=2.12
ARG SHA512HASH="33E5E009792D2730F74223B29EBA7C7DF1D966F72655738923CA0B997D2370AEAA0ED6BFA8FFF222C44204321970BA3C74730028B0964F5DFC9F46A0947F2897"

ENV KAFKA_VERSION=$KAFKA_VERSION \
    SCALA_VERSION=$SCALA_VERSION \
    KAFKA_HOME=/kafka \
    SHA512HASH=$SHA512HASH \
    KAFKA_URL_PATH=kafka/$KAFKA_VERSION/kafka_$SCALA_VERSION-$KAFKA_VERSION.tgz

#
# Create a user and home directory for Kafka
#
USER root
RUN apt-get update && apt-get install iproute curl openjdk-8-jdk wget gnupg2 -y

RUN groupadd -r kafka -g 1001 &&  \
    useradd -u 1001 -r -g kafka -m -d $KAFKA_HOME -s /sbin/nologin -c "Kafka user" kafka && \
    chmod 755 $KAFKA_HOME
RUN mkdir $KAFKA_HOME/data && \
    mkdir $KAFKA_HOME/logs

#
# Change ownership and switch user
#
RUN chown -R kafka $KAFKA_HOME && \
    chgrp -R kafka $KAFKA_HOME

#
# Download Kafka
#
RUN curl -fSL -o /tmp/kafka.tgz $(curl --stderr /dev/null https://www.apache.org/dyn/closer.cgi\?as_json\=1 | \
  sed -rn 's/.*"preferred":.*"(.*)"/\1/p')$KAFKA_URL_PATH ||  \
  curl -fSL -o /tmp/kafka.tgz https://archive.apache.org/dist/$KAFKA_URL_PATH

#
# Verify the contents and then install ...
#
RUN wget -O /tmp/KEYS https://downloads.apache.org/kafka/KEYS && \
    wget -O /tmp/key.asc https://archive.apache.org/dist/kafka/$KAFKA_VERSION/kafka_$SCALA_VERSION-$KAFKA_VERSION.tgz.asc && \
    gpg --import /tmp/KEYS && \
    gpg --verify /tmp/key.asc /tmp/kafka.tgz

RUN echo "$SHA512HASH /tmp/kafka.tgz" | sha512sum -c - && \
    tar -xzf /tmp/kafka.tgz -C $KAFKA_HOME --strip-components 1 && \
    rm -f /tmp/kafka.tgz

# Remove unnecessary files
RUN rm -f $KAFKA_HOME/libs/*-{sources,javadoc,scaladoc}.jar* && \
    rm -r $KAFKA_HOME/site-docs

RUN chmod -R g+w,o+w $KAFKA_HOME
USER kafka

WORKDIR $KAFKA_HOME
EXPOSE 9092

