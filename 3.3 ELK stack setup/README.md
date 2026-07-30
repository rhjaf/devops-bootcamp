## ELK stack setup
In this task, we will setup a ELK (Elastic, Logstash, Kibana) stack.
1. Execute the below command to setup containers
    ```bash
    docker compose up -d
    ```
    the docker compose content is as below
    ```yaml
    services:
        elasticsearch:
            image: nexus.local:8082/elasticsearch:9.4.4
            container_name: es
            restart: unless-stopped

            environment:
                - discovery.type=single-node
                - xpack.security.enabled=false
            volumes:
                - es_data:/usr/share/elasticsearch/data
                - ./elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
            networks:
                - elk-net
            ports:
                - "9200:9200"

        logstash:
            image: nexus.local:8082/logstash:9.4.4
            container_name: ls
            restart: unless-stopped
            depends_on:
                - elasticsearch
            environment:
                - LS_JAVA_OPTS=-Xms512m -Xmx512m
            volumes:
                - ls_data:/usr/share/logstash/data
                - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
                #- ./logstash_jvm.options:/usr/share/logstash/config/jvm.options:ro
                #- ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
                #  - ./logstash_pipeline:/usr/share/logstash/pipeline:ro
            ports:
                - "5044:5044"
            networks:
                - elk-net

        kibana:
            image: nexus.local:8082/kibana:9.4.4
            container_name: ki
            restart: unless-stopped

            depends_on:
                - elasticsearch
            environment:
                - ELASTICSEARCH_HOSTS=http://es:9200

            ports:
                - "5601:5601"

            volumes:
                - ki_data:/usr/share/kibana/data

            networks:
                - elk-net
    volumes:
        es_data:
        ls_data:
        ki_data:

    networks:
        elk-net:
            driver: bridge

    ```
    the `elasticsearch.yml` is as below
    ```yml
    cluster.name: "elk0"
    network.host: 0.0.0.0
    xpack.license.self_generated.type: basic
    cluster.max_shards_per_node: 2000
    ```
    the `logstash.conf` is as below
    ```yaml
    input {
        beats {
            port => 5044
        }
    }

    filter {
    }

    output {
        elasticsearch {
            hosts => ["http://es:9200"]
            index => "logs-%{+YYYY.MM.dd}"
        }
    }
    ```
2. Now we can connect to Kibana dashboard via `192.168.1.128:5601`