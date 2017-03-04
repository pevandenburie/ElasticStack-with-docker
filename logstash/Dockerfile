#FROM logstash:5
FROM docker.elastic.co/logstash/logstash:5.2.2

#RUN logstash-plugin install logstash-filter-de_dot
RUN logstash-plugin install logstash-input-beats

COPY logstash.conf /config-dir/

CMD ["-f", "/config-dir/logstash.conf"]
