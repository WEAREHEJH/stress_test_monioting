# 📊 MySQL + SpringBoot 모니터링 시스템 구축

Spring Boot 애플리케이션과 MySQL 데이터베이스의 성능 지표를 Prometheus로 수집하고, Grafana를 통해 시각화하는 모니터링 시스템을 구축합니다.

또한 부하 테스트를 통해 실제 사용 상황을 시뮬레이션하며, 수집된 데이터를 기반으로 애플리케이션의 안정성과 성능을 분석합니다.

<br>

## 👨‍💻 팀원

| <img src="https://avatars.githubusercontent.com/u/193316939?v=4" width="220" />|<img src="https://avatars.githubusercontent.com/u/73926352?v=4" width="220" />|<img src="https://avatars.githubusercontent.com/u/193316939?v=4" width="220"/>|<img src="https://github.com/eundeom.png" width="220" /> |
|:-:|:-:|:-:|:-:|
|박재희<br/>[@JaeHee-devSpace](https://github.com/JaeHee-devSpace) |임하은<br/>[@imhaeunim](http://github.com/imhaeunim)|박정호<br/>[@Jeongho427](https://github.com/Jeongho427)|이은정<br/>[@eundeom](https://github.com/eundeom) |

<br>
<br>

## 🎯 프로젝트 목표

✔️ **MySQL을 Prometheus에 연결**하여 쿼리 수, 연결 수 등 주요 지표를 수집 및 시각화

✔️ **Spring Boot 애플리케이션을 Prometheus에 연동**하여 JVM 메모리, HTTP 요청, GC 등 애플리케이션 성능 지표를 수집

✔️ **Grafana를 통해 Prometheus에서 수집된 메트릭을 실시간으로 대시보드로 시각화**

✔️ **Ubuntu 재부팅 후에도 모든 Exporter와 모니터링 구성 요소가 자동 실행**되도록 systemd 기반 설정 적용

✔️ **HTTP 요청 부하 및 CPU 부하 테스트를 통해 수집된 메트릭의 반응을 검증**

<br>
<br>

--- 

## 📚 목차

1. [mysqld_exporter 설정](#1%EF%B8%8F⃣-mysqld_exporter-설정)
2. [Spring Boot 구성](#2%EF%B8%8F⃣-spring-boot-구성)
3. [Prometheus 연결](#3%EF%B8%8F⃣-prometheus-연결)
4. [Grafana 설정](#4%EF%B8%8F⃣-grafana-설정)
5. [부하 테스트](#5%EF%B8%8F⃣-부하-테스트)
6. [PromQL 예시](#6%EF%B8%8F⃣-기타-promql-예시)

<br>
<br>

--- 


## ⚙️ 아키텍처

<p>
  <img src="https://img.shields.io/badge/ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white">
  <img src="https://img.shields.io/badge/springboot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white">
  <img src="https://img.shields.io/badge/mysql-4479A1?style=for-the-badge&logo=mysql&logoColor=white">
  <img src="https://img.shields.io/badge/prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white">
  <img src="https://img.shields.io/badge/grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white">
  <img src="https://img.shields.io/badge/docker-2496ED?style=for-the-badge&logo=docker&logoColor=white">
</p>
<img src="https://github.com/user-attachments/assets/08886f2e-5615-4956-a9fe-156e157129db" width="900" />



<br>
<br>

--- 

## 1️⃣ mysqld_exporter 설정

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

<br>

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

<br>
<br>

---

## 2️⃣ Spring Boot 구성

### `build.gradle` 설정

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'io.micrometer:micrometer-registry-prometheus'
```
<br>

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
<br>

### Dockerfile 예시

```
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY . /app
CMD ["java", "-jar", "step07_cicd-0.0.7-SNAPSHOT.jar"]
```
<br>

### SpringBoot Docker 실행

```bash
docker run -d -p 8080:8080 --name myjar eundeom/myjar
```

<br>
<br>

---

## 3️⃣ Prometheus 연결

### `/etc/prometheus/prometheus.yml`에 설정 추가

```yaml
- job_name: 'myjar'
  metrics_path: '/actuator/prometheus'
  static_configs:
    - targets: ['localhost:8080']
```
<br>

### Prometheus 재시작

```bash
sudo systemctl restart prometheus
```
![image](https://github.com/user-attachments/assets/06c8ddfb-c30e-4fc0-ab48-e322b5a2ff86)

<br>
<br>

---

## 4️⃣ Grafana 설정

### 1. Grafana Dashboard ID: **6756**

> 👉 Spring Boot Statistics Dashboard (ID: 6756)
> 

<br>

### 2. 데이터 소스 선택 및 Target 확인

- Prometheus 데이터 소스 선택
- Target IP: `localhost:8080` 또는 컨테이너 내부 IP 사용

![image](https://github.com/user-attachments/assets/2654fdd3-1f7c-4482-8432-c2f9ecd1f9ad)

<br>
<br>

---

## 5️⃣ 부하 테스트

### Spring Boot 애플리케이션 부하 (HTTP)

```bash
sudo apt install apache2-utils -y
ab -n 10000 -c 100 http://localhost:8080/
```

- `n`: 총 요청 수
- `c`: 동시 요청 수

Grafana를 통해 JVM 메모리, GC, 요청 처리량 등을 확인 가능

![image](https://github.com/user-attachments/assets/f2f9332c-3dc2-4d5e-a0b5-87828857c4df)


<br>
<br>

---

## 6️⃣ 기타 PromQL 예시

| 지표 | PromQL |
| --- | --- |
| HTTP 요청 수 | `rate(http_server_requests_seconds_count[1m])` |
| JVM 메모리 사용량 | `jvm_memory_used_bytes` |
| CPU 사용률 | `rate(process_cpu_seconds_total[1m])` |
| MySQL 쿼리 수 | `rate(mysql_global_status_queries[1m])` |
