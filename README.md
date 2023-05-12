# docker-elk-stack

![GitHub](https://img.shields.io/github/license/julianhm9612/docker-elk-stack?style=for-the-badge) ![GitHub last commit](https://img.shields.io/github/last-commit/julianhm9612/docker-elk-stack?style=for-the-badge)

Example of an elastic stack configuration and filebeat in docker :whale:

## Work flow
> log -> filebeat -> logstash -> elasticsearch <- kibana

## Previous requirements

1. Docker
2. Docker-compose

## Usage

To run the entire stack
> `docker-compose up`

To down the stack
> `docker-compose down`

## Initial configuration

### 1. Enable SSL
Create SSL certificate for Elasticsearch and enable SSL

1. bin/elasticsearch-certutil ca
2. bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
3. docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-certificates.p12 .
4. docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-stack-ca.p12 .

### 2. Create passwords for basic users
Run the next command to generate passwords for all the built-in users:
> `bin/elasticsearch-setup-passwords auto`

### 3. Create pem for logstash
1. openssl pkcs12 -in elastic-certificates.p12 -out logstash.pem -clcerts -nokeys
2. openssl pkcs12 -in elastic-certificates.p12 -cacerts -nokeys -chain | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > logstash-ca.crt
3. docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash.pem .
4. docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash-ca.crt ./es-ca.crt

### 4. Extract elk certificates from docker

> `docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/certs ./certs`

## Customize Config
- templates/custom-logs.template.json : Change it to your log index
```
# Make your own log index
{
    ...
    "mappings": {
        "properties": {
            "name": {
                "type": "keyword"
            },
            "class": {
                "type": "keyword"
            },
            "state": {
                "type": "integer"
            },
            "@timestamp": {
                "type": "date"
            }
        }
    }
}
```
- logstash.conf
```
# Change 'timestamp' to your log custom timestamp key
filter {
  ...
  date{
    match => ["timestamp", "UNIX_MS"]
    target => "@timestamp"
  }
}
```
```
# Change 'time.localtime' to your location time
filter {
  ...
  ruby {
    code => "event.set('indexDay', event.get('[@timestamp]').time.localtime('+09:00').strftime('%Y%m%d'))"
  }
}
```

## if you have errors in chrome with the certificate you can run
    sendCommand(SecurityInterstitialCommandId.CMD_PROCEED)

## Urls and ports

### Kibana
https://localhost:5601/

### Elasticsearch
https://localhost:9200/

https://localhost:9200/_cluster/health/?pretty

https://localhost:9200/_xpack

https://localhost:9200/_cat/indices?v

https://localhost:9200/_aliases

### Filebeat
http://localhost:5066/?pretty

# Task List

- [x] runing all the stack
- [x] reading data from log file
- [x] reading data from postgresql

# License

[MIT License](https://raw.githubusercontent.com/julianhm9612/docker-elk-stack/master/LICENSE)
Copyright (c) 2023 Julian Henao Marin

# Contribution

PR(s) are Open and Welcomed.
