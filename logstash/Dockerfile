FROM docker.elastic.co/logstash/logstash:7.15.1

ADD pipeline/ /usr/share/logstash/pipeline/

RUN bin/logstash-plugin install --version=3.1.5 logstash-output-mongodb
