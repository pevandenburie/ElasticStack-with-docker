Tutorial to use Kibana-ElasticSearch-Logstash from their containers.

## Install docker-compose

    $ curl -L https://github.com/docker/compose/releases/download/1.11.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    $ chmod +x /usr/local/bin/docker-compose


## Using the docker-compose file

The simpler method:

    $ docker-compose up

Connect to Kibana web UI : http://localhost:5601
Username: **elastic**
Password: **changeme**

    $ docker-compose down

Or, to remove the volumes:

    $ docker-compose down -v


## Filebeat

First download FileBeat:

https://www.elastic.co/downloads/beats/filebeat

Pushing the Filebeat template to ElasticSearch:

    $ curl -XPUT -u elastic:changeme 'http://localhost:9200/_template/filebeat' -d@filebeat.template.json

To remove old documents:

    $ curl -XDELETE 'http://localhost:9200/filebeat-*'

Import dashboard and index into ElasticSearch:

    $ ./import_dashboards -user elastic -pass changeme -es "http://localhost:9200"

Starting FileBeat:

    $ ./filebeat -c filebeat.yml -e -d '* '


## Disabling X-Pack

There is a 30-days trial period.

    xpack.security.enabled

Set to ``false`` to disable X-Pack security. Configure in both ``elasticsearch.yml`` and ``kibana.yml``.


## Example

https://www.elastic.co/guide/en/kibana/current/tutorial-load-dataset.html

Download example data file:

    $ wget https://download.elastic.co/demos/kibana/gettingstarted/shakespeare.json

Setup the mapping of the data:

    $ curl -u elastic -XPUT http://localhost:9200/shakespeare -d '
    {
     "mappings" : {
      "_default_" : {
       "properties" : {
        "speaker" : {"type": "string", "index" : "not_analyzed" },
        "play_name" : {"type": "string", "index" : "not_analyzed" },
        "line_id" : { "type" : "integer" },
        "speech_number" : { "type" : "integer" }
       }
      }
     }
    }
    ';

    $ curl -u elastic -XPUT http://localhost:9200/logstash-2015.05.18 -d '
    {
      "mappings": {
        "log": {
          "properties": {
            "geo": {
              "properties": {
                "coordinates": {
                  "type": "geo_point"
                }
              }
            }
          }
        }
      }
    }
    ';

    $ curl -u elastic -XPUT http://localhost:9200/logstash-2015.05.19 -d '
    {
      "mappings": {
        "log": {
          "properties": {
            "geo": {
              "properties": {
                "coordinates": {
                  "type": "geo_point"
                }
              }
            }
          }
        }
      }
    }
    ';

    $ curl -u elastic -XPUT http://localhost:9200/logstash-2015.05.20 -d '
    {
      "mappings": {
        "log": {
          "properties": {
            "geo": {
              "properties": {
                "coordinates": {
                  "type": "geo_point"
                }
              }
            }
          }
        }
      }
    }
    ';

Load the data:

    $ curl -u elastic:changeme -XPOST 'localhost:9200/shakespeare/_bulk?pretty' --data-binary @shakespeare.json

Verify it worked:

    $ curl -u elastic 'localhost:9200/_cat/indices?v'
    health status index                             uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    yellow open   logstash-2015.05.18               5xsXBEWeSJ-f29fbPOrVjg   5   1          0            0       650b           650b
    yellow open   .monitoring-logstash-2-2017.03.09 86rNXO0lT5e4XoB5zdPUOQ   1   1        158            0    103.8kb        103.8kb
    yellow open   filebeat-2017.03.09               AVKvVVDyTE-U9vfLF8lyiQ   5   1          3            0     28.3kb         28.3kb
    yellow open   .kibana                           wlUmA4dwT8SMXr2NVzo98Q   1   1          2            0      8.7kb          8.7kb
    yellow open   .monitoring-es-2-2017.03.09       afj5KLUUTUelEu5MLBI25w   1   1       2331          360      1.3mb          1.3mb
    yellow open   logstash-2015.05.19               n4oDJw33RKiwU2thuU0k6g   5   1          0            0       650b           650b
    yellow open   .monitoring-kibana-2-2017.03.09   ArC5Ux7ZQxSHX6qcrJVDnQ   1   1        157            0     89.2kb         89.2kb
    yellow open   .monitoring-data-2                xuqE2mIyS96CClJC9OdAhA   1   1          4            0      7.4kb          7.4kb
    yellow open   shakespeare                       0khENz1JRBCyJBTrNxmrtA   5   1     111396            0     28.7mb         28.7mb
    yellow open   logstash-2015.05.20               4OjkvDZpRAi-9k6BpLFXBw   5   1          0            0       650b           650b


## Using FileBeat in a container

https://discuss.elastic.co/t/collecting-logfiles-of-docker-containers-with-filebeat-running-as-docker-container/77548

Run in standalone:

    $ docker run -it -v ./filebeat/filebeat.yml:/filebeat.yml -v /var/lib/docker/containers:/hostfs/var/lib/docker/containers:rw prima/filebeat:5.2.2


## Redirect the logs of a container

http://stackoverflow.com/questions/33432983/docker-apps-logging-with-filebeat-and-logstash

Regarding your hint about the GUIDs, I agree, however you probably would not want to make configuration like this by hand, but rather use something like Ansible. Then just "docker ps | grep container_name | awk '{print $1}'", then template the result into the config and restart filebeat.

http://eembsen.github.io/2015/12/05/docker-es-filebeat/


# Elastic errors

https://www.elastic.co/guide/en/elasticsearch/reference/5.2/docker.html

https://github.com/elastic/elasticsearch/issues/19987
https://github.com/elastic/elasticsearch/issues/19868

    elasticsearch_1  | [2017-03-17T21:36:45,592][INFO ][o.e.n.Node               ] initialized
    elasticsearch_1  | [2017-03-17T21:36:45,596][INFO ][o.e.n.Node               ] [k8kIPoW] starting ...
    elasticsearch_1  | [2017-03-17T21:36:46,752][WARN ][i.n.u.i.MacAddressUtil   ] Failed to find a usable hardware address from the network interfaces; using random bytes: 3c:2b:0e:fc:97:03:a2:3d
    elasticsearch_1  | [2017-03-17T21:36:47,090][INFO ][o.e.t.TransportService   ] [k8kIPoW] publish_address {172.18.0.2:9300}, bound_addresses {[::]:9300}
    elasticsearch_1  | [2017-03-17T21:36:47,122][INFO ][o.e.b.BootstrapChecks    ] [k8kIPoW] bound or publishing to a non-loopback or non-link-local address, enforcing bootstrap checks
    elasticsearch_1  | ERROR: bootstrap checks failed
    elasticsearch_1  | max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
    elasticsearch_1  | [2017-03-17T21:36:47,215][INFO ][o.e.n.Node               ] [k8kIPoW] stopping ...
    elasticsearch_1  | [2017-03-17T21:36:47,290][INFO ][o.e.n.Node               ] [k8kIPoW] stopped
    elasticsearch_1  | [2017-03-17T21:36:47,296][INFO ][o.e.n.Node               ] [k8kIPoW] closing ...
    elasticsearch_1  | [2017-03-17T21:36:47,358][INFO ][o.e.n.Node               ] [k8kIPoW] closed


    $ ssh -i ~/Box\ Sync/keys/paulvand-key01.pem cloud-user@10.203.49.168

    $ sudo sysctl -w vm.max_map_count=262144


        elasticsearch_1  | [2017-03-17T21:48:21,195][WARN ][i.n.u.i.MacAddressUtil   ] Failed to find a usable hardware address from the network interfaces; using random bytes: fe:cf:30:57:eb:24:34:7f
        elasticsearch_1  | [2017-03-17T21:48:21,818][INFO ][o.e.t.TransportService   ] [k8kIPoW] publish_address {172.18.0.2:9300}, bound_addresses {[::]:9300}
        elasticsearch_1  | [2017-03-17T21:48:21,892][INFO ][o.e.b.BootstrapChecks    ] [k8kIPoW] bound or publishing to a non-loopback or non-link-local address, enforcing bootstrap checks
        elasticsearch_1  | ERROR: bootstrap checks failed
        elasticsearch_1  | max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
        elasticsearch_1  | [2017-03-17T21:48:21,957][INFO ][o.e.n.Node               ] [k8kIPoW] stopping ...
        elasticsearch_1  | [2017-03-17T21:48:22,120][INFO ][o.e.n.Node               ] [k8kIPoW] stopped
        elasticsearch_1  | [2017-03-17T21:48:22,123][INFO ][o.e.n.Node               ] [k8kIPoW] closing ...
        elasticsearch_1  | [2017-03-17T21:48:22,252][INFO ][o.e.n.Node               ] [k8kIPoW] closed
        elasticstackwithdocker_elasticsearch_1 exited with code 78


# Kafka

https://www.elastic.co/blog/just-enough-kafka-for-the-elastic-stack-part1


# Issues using docker

filebeat_1       | 2017/03/24 21:54:09.301520 sync.go:85: ERR Failed to publish events caused by: write tcp 172.18.0.5:41108->172.18.0.4:5044: write: connection reset by peer
filebeat_1       | 2017/03/24 21:54:09.301603 single.go:91: INFO Error publishing events (retrying): write tcp 172.18.0.5:41108->172.18.0.4:5044: write: connection reset by peer
