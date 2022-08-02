# Kafka 실험

## 목적
특정 유저 및 그룹 권한으로 broker & zookeeper 컨테이너 실행 
해당 유저 권한의 os 디렉토리로 볼륨 바인딩하기
도커 컨테이너 삭제 후 생성 (docker-compose down & up) 해도 볼륨의 meta.properties 파일로 에러가 발생하지 않기

## 환경 & 의존성
- mac os
- docker version : Docker version 20.10.10, build b485636
- docker-compose version : docker-compose version 1.29.2, build 5becea4c
- zookeeper : confluentinc/cp-zookeeper:7.0.1
- kafka : confluentinc/cp-kafka:7.0.1

## 이미지 변경
- Official Dockerfile 참고
    - [zookeeper](https://github.com/confluentinc/cp-docker-images/blob/master/debian/zookeeper/Dockerfile)
    - [kafka](https://github.com/confluentinc/cp-docker-images/blob/master/debian/kafka/Dockerfile)
- 특정 유저로 컨테이너 생성시 에러가 발생하는 지점
    - /var/lib/zookeeper 나 /var/lib/kafka 에 쓰기가 가능한지 확인
    - /etc/kafka 가 실행 가능한 지 확인 (정확치 않음)
- 최종
    - 컨테이너 생성시 권한 관련 에러가 발생하는 디렉토리들의 권한을 바꿔줌 (uid:gid, 770)
    - 그 외, 볼륨 바인딩 하진 않지만 확인해볼 수도 있는 디렉토리들도 편의상 권한을 바꿔줌


## 그외 참고 1 [Required Config](https://docs.confluent.io/platform/current/installation/docker/config-reference.html)
- ZOOKEEPER_CLIENT_PORT : Instructs ZooKeeper where to listen for connections by clients such as Apache Kafka®.
- KAFKA_ZOOKEEPER_CONNECT : Instructs Kafka how to get in touch with ZooKeeper.
- KAFKA_ADVERTISED_LISTENERS : Describes how the host name that is advertised and can be reached by clients. The value is published to ZooKeeper for clients to use.
- KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: set to 1 when you are running with a single-node cluster 
- KAFKA_BROKER_ID 를 지정해주지 않으면, 컨테이너를 삭제했다 올릴 때, 볼륨의 meta.metaproperties broker.id 값이 새로 생성하는 컨테이너의 broker.id 값과 일치하지 않아 에러가 발생함

## 그외 참고 2 volumes
도커컴포즈를 띄우면 총 다섯 개의 도커 볼륨이 생기는 데,
만약 os 특정 경로에서 사용하려면 각각 바인딩 시켜주면 된다
주의) 상위에서 묶으면 바인딩 되지 않는다 (./zookeeper:/var/lib/zookeeper -> x)

- /var/lib/zookeeper/log
- /var/lib/zookeeper/data
- /var/lib/kafka/data
- /etc/kafka/secrets (연결을 암호화할 때 사용)
- /etc/zookeeper/secrets (연결을 암호화할 때 사용)

## 실행 방법
```bash
mkdir -p ./data/kafka_data ./data/zk_data/data ./data/zk_data/log
docker build -t kafka:latest -f ./Dockerfile_kafka .
docker build -t zookeeper:latest -f ./Dockerfile_zk .
docker-compose up -d
```
주의) 볼륨 바인딩 된 디렉토리들의 유저가 도커컴포즈, 도커 파일의 유저가 모두 일치해야함