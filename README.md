Tutorial to use Kibana-ElasticSearch-Logstash from their containers.

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
