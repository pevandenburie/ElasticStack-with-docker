Tutorial to use Kibana-ElasticSearch-Logstash from their containers.


## ElasticSearch

https://www.elastic.co/guide/en/beats/libbeat/current/elasticsearch-installation.html

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html


    $ docker pull docker.elastic.co/elasticsearch/elasticsearch:5.2.2
    $ docker run --rm -d -p 9200:9200 -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" docker.elastic.co/elasticsearch/elasticsearch:5.2.2

    $ curl -u elastic:changeme http://localhost:9200   // default X-Pack password is "changeme"

    {
      "name" : "C34DJmz",
      "cluster_name" : "docker-cluster",
      "cluster_uuid" : "ZUEtuv5hTLeGemUT4c3KAQ",
      "version" : {
        "number" : "5.2.2",
        "build_hash" : "f9d9b74",
        "build_date" : "2017-02-24T17:26:45.835Z",
        "build_snapshot" : false,
        "lucene_version" : "6.4.1"
      },
      "tagline" : "You Know, for Search"
    }

Other methods:

    $ docker run -d -v "$PWD/config":/usr/share/elasticsearch/config elasticsearch
This image is configured with a volume at /usr/share/elasticsearch/data to hold the persisted index data. Use that path if you would like to keep the data in a mounted volume:

    $ docker run -d -v "$PWD/esdata":/usr/share/elasticsearch/data elasticsearch
This image includes EXPOSE 9200 9300 (default http.port), so standard container linking will make it automatically available to the linked containers.

Using the docker-compose file:

    $ docker-compose up
    $ curl -u elastic http://127.0.0.1:9200/_cat/health
    $ docker-compose down -v  // To destroy the cluster **and the data volumes**



## Logstash

https://www.elastic.co/guide/en/logstash/current/docker.html

By default, the container will look in /usr/share/logstash/pipeline/ for pipeline configuration files.

    $ docker run --rm -it -v ~/pipeline/:/usr/share/logstash/pipeline/ docker.elastic.co/logstash/logstash:5.2.2

If you don’t provide configuration to Logstash, it will run with a minimal config that listens for messages from the **Beats input plugin** and echoes any that are received to stdout.


Beats plugin: https://www.elastic.co/guide/en/logstash/current/plugins-inputs-beats.html

To customize the Logstash docker image to include plugins and config file:



    $ docker build -t logstash-beats .
    $ docker run -d my-logstash


Other methods:

    $ docker pull logstash     // to be deprecated
    $ docker run -it --rm logstash -e 'input { stdin { } } output { stdout { } }'
    $ docker run -it --rm -v "$PWD":/config-dir logstash -f /config-dir/logstash.conf
