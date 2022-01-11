https://blog.codefarm.me/2018/06/29/elasticsearch-fluentd-kibana-docker-compose/

```yml
version: "2.4"
services:
    elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch-oss:7.10.2
        restart: on-failure
        mem_limit: 2g
        environment:
          - discovery.type=single-node
        ports:
          - 9200
        volumes:
          - /var/lib/elasticsearch:/usr/share/elasticsearch/data
        networks:
          - local
        depends_on:
          - fluent-bit
        logging:
          driver: fluentd
          options:
            tag: efk.es
    kibana:
        image: docker.elastic.co/kibana/kibana-oss:7.10.2
        restart: on-failure
        mem_limit: 256m
        environment:
          - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
        ports:
          - 5601:5601
        networks:
          - local
        depends_on:
          - fluent-bit
          - elasticsearch
        logging:
          driver: fluentd
          options:
            tag: efk.kibana
    fluent-bit:
        image: fluent/fluent-bit:1.8
        command:
          - /fluent-bit/bin/fluent-bit
          - --config=/etc/fluent-bit/fluent-bit.conf
        environment:
            - FLB_ES_HOST=elasticsearch
            - FLB_ES_PORT=9200
        ports:
          - 2020:2020
          - 24224:24224
        volumes:
          - ./conf/:/etc/fluent-bit/:ro
        networks:
          - local
        logging:
          driver: fluentd
          options:
            tag: efk.fluent-bit
networks:
  local:
    driver: bridge
```
