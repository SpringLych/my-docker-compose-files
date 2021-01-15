ARG ELK_VERSION

# https://www.docker.elastic.co/
#FROM docker.elastic.co/logstash/logstash:${ELK_VERSION}
FROM logstash:7.9.3

COPY connect/mysql-connector-java-8.0.17.jar /usr/share/logstash/mysql-connector.jar
# RUN bin/logstash-plugin install logstash-input-jdbc