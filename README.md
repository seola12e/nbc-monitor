## 모니터링 시스템 및 시큐어 코딩

### 1-4 Prometheus

`sample`과 동일한 위치에 `prometheus` 디렉토리 생성 후 아래와 같이 `prometheus.yml` 작성

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-boot'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

이후 아래 `/path/to` 부분을 경로에 맞게 치환한 후, 해당 docker 명령어로 Prometheus 실행
```commandline
docker run -d --name=prometheus -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```


### 1-5 Grafana

아래 docker 명령어로 Grafana를 3000번 포트에서 실행하도록 함
```commandline
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```
[localhost:3000](http://localhost:3000) 에 접속 시 아이디와 비밀번호를 입력하라고 뜨는데, 따로 설정해 놓지 않으면 기본적으로 둘 다 `admin`


### 1-7 애플리케이션 로그 모니터링

`sample`과 동일한 위치에 `loki` 디렉토리 생성 후 아래와 같이 `loki-config.yml` 작성

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  instance_addr: 127.0.0.1
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

schema_config:
  configs:
    - from: 2020-10-24
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
```

이후 아래 `${loki-config.yml 이 저장된 폴더}` 부분을 경로에 맞게 치환한 후, 해당 docker 명령어로 Loki 실행
```commandline
docker run --name loki -d -v ${loki-config.yml 이 저장된 폴더}:/mnt/config -p 3100:3100 grafana/loki:3.0.0 -config.file=/mnt/config/loki-config.yml
```