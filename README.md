# docker-elk-stack

![Elastic stack](https://img.shields.io/badge/Elastic%20Stack-8.4.3-blue?style=for-the-badge&logo=elasticsearch)
![GitHub](https://img.shields.io/github/license/julianhm9612/docker-elk-stack?style=for-the-badge)
![GitHub last commit](https://img.shields.io/github/last-commit/julianhm9612/docker-elk-stack?style=for-the-badge)

## Introduction
Example of an elastic stack configuration in docker :whale:

## Work flow
> log -> filebeat -> logstash -> elasticsearch <- kibana

## Main Features
- Security enabled by default.
- Example of reading data from log file.
- Example of reading postgresql data.

## Requirements
- [Docker 20.05 or higher](https://docs.docker.com/install/)
- [Docker-Compose 1.29 or higher](https://docs.docker.com/compose/install/)
- 4GB RAM (For Windows and MacOS make sure Docker's VM has more than 4GB+ memory.)

## Initial configuration

### 1. Run elastic search container
> docker run -d --name elasticsearch elasticsearch:8.4.3

### 2. Running an Interactive Shell in a elastic search Container
> docker exec -it elasticsearch sh

### 3. Create a directory called certs and enter
> mkdir certs && cd certs

### 4. Enable SSL
Create a self-signed certificate for Elasticsearch

> /usr/share/elasticsearch/bin/elasticsearch-certutil ca

> /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --dns elasticsearch,logstash

> /usr/share/elasticsearch/bin/elasticsearch-certutil cert --pem -ca elastic-stack-ca.p12 --dns kibana

> openssl pkcs12 -in elastic-certificates.p12 -out logstash.pem -clcerts -nokeys

> openssl pkcs12 -in elastic-certificates.p12 -nocerts -nodes | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > logstash-ca.key

> openssl pkcs12 -in elastic-certificates.p12 -cacerts -nokeys -chain | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > logstash-ca.crt

> openssl pkcs12 -in elastic-certificates.p12 -clcerts -nokeys | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > logstash.crt

> /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca-cert logstash-ca.crt --ca-key logstash-ca.key --dns logstash --pem

> openssl pkcs8 -in logstash-ca.key -topk8 -nocrypt -out logstash.pkcs8.key

### 5. Get out of the container
> exit

### 6. Extract elk certificates from docker
> docker cp elasticsearch:/usr/share/elasticsearch/certs ./certs

### 7. Enable SSL and TLS
> /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password

> /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

> /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password

> /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password

### 8. Create passwords for basic users
Run the next command to generate passwords for all the built-in users:
> bin/elasticsearch-setup-passwords auto

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

## Usage
To run the entire stack
> docker-compose up

To down the stack
> docker-compose down

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

- [] Automatic self-signed certificate generation
- [] 
- [] 

# License

[MIT License](https://raw.githubusercontent.com/julianhm9612/docker-elk-stack/master/LICENSE)
Copyright (c) 2023 Julian Henao Marin

# Contribution

PR(s) are Open and Welcomed.
