# Steps

## Generate elk certificates

1.
       /usr/share/elasticsearch/bin/elasticsearch-certutil ca
2.
       /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --dns elasticsearch,logstash

-- /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password

-- /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

-- /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password

-- /usr/share/elasticsearch/bin/elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password

3.
       /usr/share/elasticsearch/bin/elasticsearch-certutil cert --pem -ca elastic-stack-ca.p12 --dns kibana

4.
       openssl pkcs12 -in elastic-certificates.p12 -out logstash.pem -clcerts -nokeys

5.
       openssl pkcs12 -in elastic-certificates.p12 -nocerts -nodes | sed -ne '/-BEGIN PRIVATE KEY-/,/-END PRIVATE KEY-/p' > logstash-ca.key

6.
       openssl pkcs12 -in elastic-certificates.p12 -cacerts -nokeys -chain | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > logstash-ca.crt

7.
       openssl pkcs12 -in elastic-certificates.p12 -clcerts -nokeys | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > logstash.crt

8.
       /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca-cert logstash-ca.crt --ca-key logstash-ca.key --dns logstash --pem

9.
       openssl pkcs8 -in logstash-ca.key -topk8 -nocrypt -out logstash.pkcs8.key

## Extract elk certificates from docker

1.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/certificate-bundle-logstash.zip ./certs

2.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/certificate-bundle.zip ./certs

3.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-certificates.p12 ./certs

4.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/elastic-stack-ca.p12 ./certs

5.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash-ca.crt ./certs

6.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash-ca.key ./certs

7.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash.crt ./certs

8.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash.pem ./certs

9.      docker cp "$(docker-compose ps -q elasticsearch)":/usr/share/elasticsearch/logstash.pkcs8.key ./certs
