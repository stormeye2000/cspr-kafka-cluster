FROM ubuntu:16.04 as downloads

ENV KAFKA_VERSION 3.2.1
ENV KAFKA_HOME /usr/local/kafka

WORKDIR /downloads

RUN apt-get update && apt-get install -y \
    wget \
    openjdk-8-jdk \
    curl

RUN wget https://downloads.apache.org/kafka/$KAFKA_VERSION/kafka-$KAFKA_VERSION-src.tgz && \
    tar -xzf kafka-$KAFKA_VERSION-src.tgz && \
    mv kafka-$KAFKA_VERSION-src $KAFKA_HOME

RUN wget -O apache_hashes https://downloads.apache.org/kafka/$KAFKA_VERSION/kafka-$KAFKA_VERSION-src.tgz.sha512
RUN find kafka-$KAFKA_VERSION-src.tgz -type f -exec sha256sum {} \; >> hashes
RUN cat hashes
RUN sha512sum apache_hashes | awk '{print $1}'

WORKDIR $KAFKA_HOME
RUN ./gradlew jar -PscalaVersion=2.13.6

WORKDIR $KAFKA_HOME/bin


