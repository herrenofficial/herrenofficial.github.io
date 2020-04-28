---
layout: post
author: yun
title: Prometheus + Grafana로 모니터링하기 (1)
date : 2020/04/26
avatar: "assets/img/profile/yun.jpg"
categories : [DevOps]
description: 최근에 프로젝트 전반에 걸쳐 모니터링 시스템을 개편하면서 사용하게 된 Prometheus와 Grafana에 대해 소개해보려 합니다!
comments: true
tags: 
- Prometheus
- Grafana
- DevOps
---

## 목차
- TOC
{:toc}

# Prometheus + Grafana로 모니터링하기 (1)

안녕하세요, 헤렌 인스타차트 팀의 윤입니다. 최근에 프로젝트 전반에 걸쳐 모니터링 시스템을 개편하면서 사용하게 된 Prometheus와 Grafana에 대해 소개해보려 합니다! 

## Prometheus와 Grafana에 대하여
### Prometheus
![](/assets/img/post/prometheus-with-grafana-1/aFDi8ir.png)

Prometheus는 간단하게 정의해보자면 **시계열 데이터베이스 및 수집도구**입니다.

주기적으로 수집 대상의 Endpoint에 HTTP Request를 날려 데이터를 수집하고 내부 데이터베이스에 적재하는 *Pull system* 이죠.

매 순간의 수치를 가져와서 timestamp를 붙여주고 관리해주는 시스템이다보니, 흘러가는 데이터들을 가져와서 수집하기 용이합니다.

![](/assets/img/post/prometheus-with-grafana-1/sE5YzSr.png)
> 데이터들은 지나간 것은 지나간대로 냅두면 안됩니다...

그리고 PromQL이라는 자체 쿼리 언어로 손쉽게 시계열 데이터를 추출하고 커스터마이즈 할 수 있습니다. 정말 시계열 데이터 처리에 최적화되어 있기 때문에 복잡한 시계열 데이터를 원하는대로 변형할 수 있습니다.


### Grafana
![](/assets/img/post/prometheus-with-grafana-1/CJG7gjy.png)

Prometheus를 비롯해 다양한 데이터소스로부터 (주로) 시계열 데이터를 가져와 시각화하고 대시보드를 만들어주는 도구입니다.

다양한 시각화 기능과, 데이터소스, 강력한 Alarm 및 Notification 기능을 제공합니다.

Web GUI 클릭 몇 번만으로 AWS CloudWatch 부럽지 않은 Alarm/Notification 기능을 사용할 수 있습니다!


## Prometheus와 Grafana 설치하기
### Prometheus
Docker로 실행해보겠습니다.

[https://prometheus.io/docs/prometheus/latest/installation/#using-docker](https://prometheus.io/docs/prometheus/latest/installation/#using-docker)
```shell
docker run \
    -p 9090:9090 \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```


prometheus.yml을 작성하는 부분은 뒤에서 다룰테니 일단 `touch` 하고 `docker run`을 해주세요.

그리고 `http://localhost:9090`에 들어가면 다음과 같은 화면을 볼 수 있습니다.

![](/assets/img/post/prometheus-with-grafana-1/yQsgruv.png)


prometheus.yml을 작성하지 않았으니 당연히 Targets에 들어가도 아무 것도 없습니다.

![](/assets/img/post/prometheus-with-grafana-1/zy5AXP6.png)


Grafana까지 마저 설치하고 prometheus.yml에서 어떤 exporter의 데이터를 가져올 지 작성해봅시다.

### Grafana
그리고 Prometheus의 데이터를 시각화해줄 Grafana를 실행해보겠습니다.
```shell
docker run -d -p 3000:3000 grafana/grafana
```

그리고 `http://localhost:3000`에 들어가면 바로 메인화면을 볼 수 있습니다

![](/assets/img/post/prometheus-with-grafana-1/8Sx6j1f.png)

기본 비밀번호는 `admin`/`admin`입니다.

이제 grafana에서 prometheus의 데이터를 가져올 수 있도록 Datasource로 prometheus를 추가해봅시다.

![](/assets/img/post/prometheus-with-grafana-1/gYEkinn.png)

![](/assets/img/post/prometheus-with-grafana-1/3smD3bG.png)

![](/assets/img/post/prometheus-with-grafana-1/5KsHusg.png)

> mac 환경에서 실행해보시는 분들께서는 prometheus의 host를 `host.docker.internal`로 바꿔서 시도해보세요!

![](/assets/img/post/prometheus-with-grafana-1/zI90SpG.png)

grafana에서 prometheus로 잘 접근하게 된 모습입니다.
이제 실제 데이터를 가져와서 grafana에서 시각화 해봅시다.


## node_exporter로 인스턴스 메트릭 수집하기
### exporter
Prometheus와 Grafana는 모두 설치했습니다.

그런데 수집할 데이터가 없네요? Prometheus가 수집할 수 있도록 그 순간의 metric을 Web server로 제공해주는 컴포넌트를 `exporter`라고 부릅니다. 

인스턴스의 시스템, 커널 정보를 수집할 수 있도록 node_exporter를 설치해봅시다.

### 설치 및 설정
1. 로컬 인스톨
    * node_exporter [공식 릴리즈 페이지](https://github.com/prometheus/node_exporter/releases)에서 Binary asset의 주소를 복사하신 다음과 같이 실행하실 수 있습니다.
    ```shell
    wget https://github.com/prometheus/node_exporter/releases/download/v*/node_exporter-*.*-amd64.tar.gz
    tar xvfz node_exporter-*.*-amd64.tar.gz
    cd node_exporter-*.*-amd64
    ./node_exporter
    ```

2. 접속 확인
    * node_exporter를 실행하신 다음 `http://localhost:9100`으로 들어가시면 이렇게 보입니다.

    ![](/assets/img/post/prometheus-with-grafana-1/68n7cip.png)
    ![](/assets/img/post/prometheus-with-grafana-1/JVaTHXX.png)


3. prometheus.yml에 추가
    * 이제 prometheus에서 node_exporter의 데이터를 수집해갈 수 있도록 설정해보겠습니다.
        아까 volume으로 연결해놓은 prometheus.yml에 다음과 같이 작성해봅시다.

    ```yaml
    scrape_configs:
      - job_name: 'node_exporter'
        static_configs:
          - targets: ['localhost:9100']
    ```

    > mac 환경에서 실행해보시는 분들께서는 target의 주소를 `host.docker.internal`로 바꿔서 시도해보세요!

    그러면 다음과 같이 정상적으로 scraping하는 것을 볼 수 있습니다.
    
    ![](/assets/img/post/prometheus-with-grafana-1/W8HtevU.png)
    
    메인화면에서는 PromQL을 바로 실행해 볼 수 있습니다.
    
    ![](/assets/img/post/prometheus-with-grafana-1/pYxlL0l.png)
    잘 동작하네요!

4. Grafana에서 대시보드 만들어보기
![](/assets/img/post/prometheus-with-grafana-1/wiVzqlI.png)

    PromQL로 쿼리한 결과를 Grafana에서 시각화 할 수 있습니다!
    Bar, Gauge, Single stat, Heatmap 등 다양한 방법으로 데이터를 나타낼 수 있습니다.
    
    ![](/assets/img/post/prometheus-with-grafana-1/qfs1xAy.png)
    ![](/assets/img/post/prometheus-with-grafana-1/SULL0a4.png)
    
    PromQL의 유연한 기능 덕분에 매우 손쉽게 시계열 평균을 구할 수 있었습니다. 
    
    사용한 코드는 다음과 같습니다:
    ```
    avg by (exported_job) (node_memory_MemFree_bytes{job="pushgateway", exported_job="crew_prod"})
    ```


## rabbitmq_exporter로 rabbitmq 수집하기
### Custom exporter
가장 유명한 exporter인 node_exporter를 위에서 사용해봤습니다.
위에서 보셨다시피, 단순히 HTTP로 접속해서 plain text를 리턴해주는게 exporter의 일이기 때문에 손쉽게 원하는 정보를 exporter로 prometheus에게 제공해줄 수 있습니다.
이를 위해서 Go / Java(Scala) / Python 등의 다양한 언어로 Client library를 제공합니다.

헤렌에서는 Celery broker로 RabbitMQ를 사용하는데, 역시 prometheus로 RabbitMQ의 자세한 지표들을 모니터링하고 있습니다.

### kbudde/rabbitmq_exporter

[https://github.com/kbudde/rabbitmq_exporter](https://github.com/kbudde/rabbitmq_exporter)

해당 레포지토리의 README대로 따라해봅시다.

1. Start rabbitMQ
```
 docker run -d -e RABBITMQ_NODENAME=my-rabbit --name my-rabbit -p 9419:9419 rabbitmq:3-management
```
2. Start rabbitmq_exporter in container.
```
 docker run -d --net=container:my-rabbit kbudde/rabbitmq-exporter
```

이렇게 rabbitmq-exporter container를 --net 옵션을 통해 rabbitmq container와 연결시켜주는 것으로 간단하게 rabbitmq의 metric을 export할 수 있습니다.

![](/assets/img/post/prometheus-with-grafana-1/Rcu0xqF.png)

![](/assets/img/post/prometheus-with-grafana-1/u4XJFN8.png)

node_exporter와 똑같은 방식으로, localhost:9419에 접속하면 이렇게 RabbitMQ의 metric을 볼 수 있습니다.

### docker-compose
저희의 rabbitmq 인스턴스는 docker-compose를 통해 실행되고 있었습니다.
그래서 docker cli의 --net option과 동일하게 동작하는 옵션을 찾아서 다음과 같이 적용했습니다.
```yaml
version: '3'

services:
  rabbitmq:
    image: <customized-rmq-image>
    container_name: rabbitmq
    hostname: rabbitmq
    network_mode: bridge
    ports:
      - "15672:15672"
      - "5672:5672"
      - "9419:9419"
  rabbitmq-exporter:
    depends_on:
      - rabbitmq
    image: kbudde/rabbitmq-exporter
    container_name: rabbitmq-exporter
    network_mode: service:rabbitmq ## << Here!
```

### In Use
![](/assets/img/post/prometheus-with-grafana-1/3mTda1s.png)

이런 식으로 RabbitMQ의 queue에 message가 몇 개 쌓였는 지를 수집할 수 있습니다.


---

이 이외에도 
 * pushgateway로 inbound가 막힌 인스턴스 메트릭 수집하기
 * Grafana Alert 활용하기
 * Custom exporter 만들어보기

같은 내용이 남아있는데, 2부에서 다루도록 하겠습니다 :)

여러분들도 Prometheus와 Grafana로 든든하게 모니터링 해보시는건 어떨까요?
