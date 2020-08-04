# Prometheus + Grafana (Redhat)

## 0. Pull docker image
```sh
docker pull prom/prometheus
docker pull grafana/grafana

mkdir prometheus
cd prometheus
mkdir prometheus_data grafana_data
```

## 1. Create prometheus yml
```yml
global:
  scrape_interval:     15s 
  evaluation_interval: 15s 

scrape_configs:
  - job_name: 'Controller'
    metrics_path: '/actuator/prometheus'
    scrape_interval: 5s
    static_configs:
    - targets: ['172.25.2.105:5080', '172.25.2.105:6080', '172.25.2.105:7080']
```

## 2. Set defaults.ini
```ini
allow_embedding = true
```

## 3. Create docker-compose.yml 
```yml
version: '3'
services:
  prometheus:
    container_name: prometheus
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus_data:/prometheus
    ports:
      - 9090:9090
  
  grafana:
    container_name: grafana
    image: grafana/grafana
    depends_on:
      - prometheus
    ports:
      - 9091:3000
    volumes:
      - ./defaults.ini:/usr/share/grafana/conf/defaults.ini
      - ./grafana_data:/var/lib/grafana
    restart: always
```

## 4. Run
```sh
docker-compose up -d
```

## 5. Grafana setting
  1. {host_url}:9091 접속
  2. Configure -> Data sources -> Add data source -> Prometheus 선택
  3. url: host.docker.internal:9090 입력 후 Save&Test으로 확인
  4. Create -> Import -> Upload JSON file -> jvm-micrometer_rev0.json 선택
  5. Prometheus -> 생성한 data source 선택 -> Import