# Centralised Logging

This is some sample code of how to do centralised logging. So you can get something as awesome as this: http://demo.kibana.org/

Tech used:

- [Logback logstash json encoder](https://github.com/logstash/logstash-logback-encoder): write json format to your logs so you can handle multiline logging with ease, e.g. stack traces.
- [Logstash forwarder](https://github.com/elasticsearch/logstash-forwarder): Lightweight client to forward logs, so you don't have to run the Logstash client in another jvm.
- [Logstash](http://logstash.net/): Logstash to receive your logs and dump them in elasticsearch.
- [ElasticSearch](http://elasticsearch.org/): Store all the things.
- [Kibana](http://www.elasticsearch.org/overview/kibana/): Visualization. Query all the things.
- Apache reverse proxy: Kibana needs direct access to elasticsearch, so you want to redirect that through a reverse proxy for security.

This assumes you're working on Ubuntu servers. Multiple senders and a single receiver. Receiver will run Kibana too.

## Receiver Installation

- wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add
- sudo vim /etc/apt/sources.list.d/elasticsearch.list
- add lines:
 - deb http://packages.elasticsearch.org/logstash/1.3/debian stable main
 - deb http://packages.elasticsearch.org/elasticsearch/0.90/debian stable main
- sudo apt-get update
- sudo apt-get install elasticsearch logstash
- mkdir ~/lumberjack
- Generate SSL certs with: ```$ openssl req -x509 -batch -days 365000 -nodes -newkey rsa:2048 -keyout lumberjack.key -out lumberjack.crt```

- Drop 'logstash-receiver.conf' and your generated SSL .crt and .key files in ~/lumberjack
- Drop 'upstart/logstash-receiver.conf' in /etc/init/. Change the 'deploy' user to the user you're using.
- sudo service elasticsearch start
- sudo service logstash-receiver start
- Now you're ready to receive data.

## Sender Installation

- Use the provided logback.xml in your Java/Clojure apps. See [clojure example logback integration](https://github.com/vaughnd/clojure-example-logback-integration) for details.
- Dependency for Maven
```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>2.4</version>
</dependency>
```
- Dependency for Lein
```[net.logstash.logback/logstash-logback-encoder "2.4"]```
- This will write your logs out in json format.
- Build and install [Logstash forwarder](https://github.com/elasticsearch/logstash-forwarder). I recommend keeping a .deb on hand.
- Create ~/lumberjack
- Copy the same SSL certs you generated for the receiver and 'lumberjack.conf' into this directory.
- Adjust the paths and receiving server in 'lumberjack.conf'
- Drop 'upstart/lumberjack.conf' in /etc/init/. Change the 'deploy' user to the user you're using.
- sudo service lumberjack start
- Your logs should be sent to the receiver now.

## Kibana Installation

This is a bit tricky. Kibana needs direct access to ElasticSearch. Logstash includes Kibana, so you can run it with 'java -jar logstash.jar web'. Or since Kibana is just javascript, dump it in your web server and configure a reverse proxy.

I've included a basic apache config as 'apache.conf' which you can drop in /etc/apache2/sites-available and enable.
This uses SSL and HTTP basic auth on everything. /elasticsearch path reverse proxies to elasticsearch running on port 9200.
And /kibana loads Kibana from /var/www/kibana.
