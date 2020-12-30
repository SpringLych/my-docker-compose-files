ARG ELK_VERSION

# https://www.docker.elastic.co/
#FROM docker.elastic.co/logstash/logstash:${ELK_VERSION}
FROM logstash:7.9.3

#RUN curl -L -o /usr/share/logstash/lib/mysql-connector-java-5.1.47.jar https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.47/mysql-connector-java-5.1.47.jar && logstash-plugin install logstash-input-jdbc


RUN logstash-plugin install logstash-input-jdbc