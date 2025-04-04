# 📊 MySQL + MySQL 모니터링 시스템 구축

Spring Boot 애플리케이션과 MySQL 데이터베이스의 성능 지표를 Prometheus로 수집하고, Grafana를 통해 시각화하는 모니터링 시스템을 구축합니다.

또한 부하 테스트를 통해 실제 사용 상황을 시뮬레이션하며, 수집된 데이터를 기반으로 애플리케이션의 안정성과 성능을 분석합니다.

## 🎯 프로젝트 목표 (Project Goals)

✅ **MySQL을 Prometheus에 연결**하여 쿼리 수, 연결 수 등 주요 지표를 수집 및 시각화

✅ **Spring Boot 애플리케이션을 Prometheus에 연동**하여 JVM 메모리, HTTP 요청, GC 등 애플리케이션 성능 지표를 수집

✅ **Grafana를 통해 Prometheus에서 수집된 메트릭을 실시간으로 대시보드로 시각화**

✅ **Ubuntu 재부팅 후에도 모든 Exporter와 모니터링 구성 요소가 자동 실행**되도록 systemd 기반 설정 적용

✅ **HTTP 요청 부하 및 CPU 부하 테스트를 통해 수집된 메트릭의 반응을 검증**

## 📚 목차

1. [⚙️ mysqld_exporter 설정](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)
2. [🌱 Spring Boot 구성](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)
3. [📡 Prometheus 연결](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)
4. [📈 Grafana 설정](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)
5. [🔥 부하 테스트](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)
6. [📌 PromQL 예시](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)
7. [✅ 정리](https://www.notion.so/1cb89533fa5880c6b6e2e1a561cf3fab?pvs=21)

## ⚙️ mysqld_exporter 설정

### systemd 서비스 등록

```bash
sudo tee /etc/systemd/system/mysqld_exporter.service <<EOF
[Unit]
Description= MySQL Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/mysqld_exporter --config.my-cnf="/etc/.my.cnf" --web.listen-address=":9104"

[Install]
WantedBy=multi-user.target
EOF

```

### 권한 설정

```bash

sudo chown prometheus:prometheus /etc/.my.cnf
sudo chmod 600 /etc/.my.cnf
```

### 실행 및 확인

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now mysqld_exporter
systemctl status mysqld_exporter
```

---

## 🌱 Spring Boot 구성

### `build.gradle` 설정

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'io.micrometer:micrometer-registry-prometheus'
```

### `application.yml` 설정

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "prometheus,health,metrics"
  prometheus:
    metrics:
      export:
        enabled: true
```

### Dockerfile 예시

```
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY . /app
CMD ["java", "-jar", "step07_cicd-0.0.7-SNAPSHOT.jar"]
```

### SpringBoot Docker 실행

```bash
docker run -d -p 8080:8080 --name myjar eundeom/myjar
```

---

## 📡 Prometheus 연결

### `/etc/prometheus/prometheus.yml`에 설정 추가

```yaml
- job_name: 'myjar'
  metrics_path: '/actuator/prometheus'
  static_configs:
    - targets: ['localhost:8080']
```

### Prometheus 재시작

```bash
sudo systemctl restart prometheus
```

---

## 📈 Grafana 설정

### 1. Grafana Dashboard ID: **6756**

> 👉 Spring Boot Statistics Dashboard (ID: 6756)
> 

### 2. 데이터 소스 선택 및 Target 확인

- Prometheus 데이터 소스 선택
- Target IP: `localhost:8080` 또는 컨테이너 내부 IP 사용

---

## 🔥 부하 테스트

### Spring Boot 애플리케이션 부하 (HTTP)

```bash
sudo apt install apache2-utils -y
ab -n 10000 -c 100 http://localhost:8080/
```

- `n`: 총 요청 수
- `c`: 동시 요청 수

Grafana를 통해 JVM 메모리, GC, 요청 처리량 등을 확인 가능

---

## 📌 기타 PromQL 예시

| 지표 | PromQL |
| --- | --- |
| HTTP 요청 수 | `rate(http_server_requests_seconds_count[1m])` |
| JVM 메모리 사용량 | `jvm_memory_used_bytes` |
| CPU 사용률 | `rate(process_cpu_seconds_total[1m])` |
| MySQL 쿼리 수 | `rate(mysql_global_status_queries[1m])` |
