# ELK Stack with Logstash for Syslog Processing

This repository provides a **Dockerized ELK Stack** (Elasticsearch, Logstash, and Kibana) setup with Logstash configured to process and filter **syslog** messages. The setup is powered by **Podman Compose**, which helps orchestrate the containers in a local environment.

## Prerequisites

- **Podman** (or Docker) installed on your machine.
- **Podman Compose** (or Docker Compose).
- Basic knowledge of **Elasticsearch**, **Logstash**, and **Kibana**.
  
## Setup

Follow these steps to set up and run the ELK stack along with Logstash for processing syslog messages.

## Configuration
The configuration files for the services (Elasticsearch, Logstash, Kibana) are already set up. Here's a breakdown of the key configuration files:

elasticsearch.yml
```
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
xpack.security.enabled: false
```
**Configures Elasticsearch to run as a single-node cluster and exposes the service on all network interfaces (0.0.0.0).**

kibana.yml

```
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://elasticsearch:9200"]
logging.verbose: true

```

**Configures Kibana to connect to the Elasticsearch service and expose the Kibana UI on all network interfaces.**

logstash.conf

```
input {
  file {
    path => "/usr/share/logstash/pipeline/syslog"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    type => "syslog"
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGBASE} %{GREEDYDATA:syslog_message}" }
      ecs_compatibility => disabled
    }

    grok {
      match => { "syslog_message" => "%{WORD:log_level} %{GREEDYDATA:message}" }
      remove_field => ["syslog_message"]
      ecs_compatibility => disabled
    }

    if [log_level] not in ["crit", "err", "warning"] {
      drop {}
    }
  }
}

output {
  if [type] == "syslog" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "syslog-logs"
    }
  }

  stdout { codec => rubydebug }
}

```

***podman-compose.yml
This file contains the definition of the services to run in Podman (or Docker) containers.***

```
version: '3.6'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.9
    container_name: elasticsearch
    restart: always
    volumes:
      - elastic_data:/usr/share/elasticsearch/data/
      - /home/user/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
    environment:
      - ES_JAVA_OPTS=-Xmx256m -Xms256m
      - discovery.type=single-node
    ports:
      - '9200:9200'
      - '9300:9300'
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:7.17.9
    container_name: logstash
    restart: always
    volumes:
      - /var/log/syslog:/usr/share/logstash/pipeline/syslog:ro
      - /home/user/logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    command: logstash -f /usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
    ports:
      - '5044:5044'
      - '9600:9600'
    environment:
      - LS_JAVA_OPTS=-Xmx256m -Xms256m
    networks:
      - elk

  kibana:
    image: docker.elastic.co/kibana/kibana:7.17.9
    container_name: kibana
    restart: always
    volumes:
      - /home/user/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
    ports:
      - '5601:5601'
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - elk

volumes:
  elastic_data:

networks:
  elk:
    driver: bridge
```

## Running the Stack

**To bring up the ELK stack, simply run:**

```
podman-compose up
```


Certainly! Here's a README.md for your setup:

markdown
Copy code
# ELK Stack with Logstash for Syslog Processing

This repository provides a **Dockerized ELK Stack** (Elasticsearch, Logstash, and Kibana) setup with Logstash configured to process and filter **syslog** messages. The setup is powered by **Podman Compose**, which helps orchestrate the containers in a local environment.

## Prerequisites

- **Podman** (or Docker) installed on your machine.
- **Podman Compose** (or Docker Compose).
- Basic knowledge of **Elasticsearch**, **Logstash**, and **Kibana**.
  
## Setup

Follow these steps to set up and run the ELK stack along with Logstash for processing syslog messages.

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/elk-stack-syslog.git
cd elk-stack-syslog
2. Configuration
The configuration files for the services (Elasticsearch, Logstash, Kibana) are already set up. Here's a breakdown of the key configuration files:

elasticsearch.yml
yaml
Copy code
cluster.name: "docker-cluster"
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
xpack.security.enabled: false
Configures Elasticsearch to run as a single-node cluster and exposes the service on all network interfaces (0.0.0.0).
kibana.yml
yaml
Copy code
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://elasticsearch:9200"]
logging.verbose: true
Configures Kibana to connect to the Elasticsearch service and expose the Kibana UI on all network interfaces.
logstash.conf
plaintext
Copy code
input {
  file {
    path => "/usr/share/logstash/pipeline/syslog"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    type => "syslog"
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGBASE} %{GREEDYDATA:syslog_message}" }
      ecs_compatibility => disabled
    }

    grok {
      match => { "syslog_message" => "%{WORD:log_level} %{GREEDYDATA:message}" }
      remove_field => ["syslog_message"]
      ecs_compatibility => disabled
    }

    if [log_level] not in ["crit", "err", "warning"] {
      drop {}
    }
  }
}

output {
  if [type] == "syslog" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "syslog-logs"
    }
  }

  stdout { codec => rubydebug }
}
Configures Logstash to read syslog files and filter out messages based on severity (crit, err, warning). The remaining logs are forwarded to Elasticsearch.
podman-compose.yml
This file contains the definition of the services to run in Podman (or Docker) containers.

yaml
Copy code
version: '3.6'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.9
    container_name: elasticsearch
    restart: always
    volumes:
      - elastic_data:/usr/share/elasticsearch/data/
      - /home/user/elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro
    environment:
      - ES_JAVA_OPTS=-Xmx256m -Xms256m
      - discovery.type=single-node
    ports:
      - '9200:9200'
      - '9300:9300'
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:7.17.9
    container_name: logstash
    restart: always
    volumes:
      - /var/log/syslog:/usr/share/logstash/pipeline/syslog:ro
      - /home/user/logstash/pipeline/logstash.conf:/usr/share/logstash/pipeline/logstash.conf:ro
    command: logstash -f /usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch
    ports:
      - '5044:5044'
      - '9600:9600'
    environment:
      - LS_JAVA_OPTS=-Xmx256m -Xms256m
    networks:
      - elk

  kibana:
    image: docker.elastic.co/kibana/kibana:7.17.9
    container_name: kibana
    restart: always
    volumes:
      - /home/user/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
    ports:
      - '5601:5601'
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - elk

volumes:
  elastic_data:

networks:
  elk:
    driver: bridge
```

Defines Elasticsearch, Logstash, and Kibana containers with their respective configurations, volumes, and network settings.

## 3. Running the Stack

To bring up the ELK stack, simply run:
```
podman-compose up
```
Or if you are using Docker Compose:

```
docker-compose up
```

This will start all the required services:

Elasticsearch on port 9200.
Logstash on port 5044 (for receiving logs) and 9600 (for monitoring).
Kibana on port 5601 (for accessing the web UI).

### 4. Accessing the Stack

Kibana UI: Open your browser and visit http://localhost:5601 to access Kibana's web interface.

Elasticsearch: You can access the Elasticsearch API directly at http://localhost:9200.
