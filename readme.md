## Docker Compose 이용 Single Broker 구성
- Docker Desktop 설치 https://docs.docker.com/desktop/mac/install/ .

- 버전확인

    ``` docker-compose version  
    Docker Compose version v2.2.3
     ```
  
- Vim docker-compose.yml 작성
  ```
    version: '3.9'
    services:
      zookeeper:
        image: confluentinc/cp-zookeeper:6.2.0
        hostname: zookeeper
        ports:
          - "2181:2181"
        environment:
          ZOOKEEPER_CLIENT_PORT: 2181
          ZOOKEEPER_TICK_TIME: 2000

      kafka:
        image: confluentinc/cp-kafka:6.2.0
        ports:
          - "9092:9092"
          - "9093:9093"
        environment:
          KAFKA_BROKER_ID: 1
          KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
          KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
          KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:9093
          KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
          KAFKA_AUTO_CREATE_TOPICS_ENABLE: "true"
          KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

    ```

- docker-compose 실행


  ```docker-compose up –d ```
- docker 상태 로그 확인하기

  ``` 
  docker ps 
  CONTAINER ID   IMAGE                             COMMAND                   CREATED         STATUS         PORTS                                        NAMES
  6c04c18ed211   confluentinc/cp-zookeeper:6.2.0   "/etc/confluent/dock…"   7 seconds ago   Up 6 seconds   2888/tcp, 0.0.0.0:2181->2181/tcp, 3888/tcp   learningspoons-zookeeper-1
  87951c24d844   confluentinc/cp-kafka:6.2.0       "/etc/confluent/dock…"   7 seconds ago   Up 6 seconds   0.0.0.0:9092-9093->9092-9093/tcp             learningspoons-kafka-1

  docker logs 6c04c18ed211

  ```

- topic 생성


  ```
   docker-compose exec kafka kafka-topics --create --topic my-topic --bootstrap-server kafka:9092 --replication-factor 1 --partitions 1 
  Created topic my-topic.
  ``` 

- topic 확인
  ``` 
  docker-compose exec kafka kafka-topics --describe --topic my-topic --bootstrap-server kafka:9092 

  Topic: my-topic TopicId: 6_AhUkRSRcyPArD474AjkQ PartitionCount: 1       ReplicationFactor: 1    Configs: 
          Topic: my-topic Partition: 0    Leader: 1       Replicas: 1     Isr: 1

  ```

- 컨슈머 실행

  ```
  docker-compose exec kafka bash 
  [appuser@94e94072e1ea ~]$ kafka-console-consumer --topic my-topic --bootstrap-server kafka:9092
  ```

- 프로듀서 실행

  ```
  docker-compose exec kafka bash 
  [appuser@94e94072e1ea ~]$ kafka-console-producer --topic my-topic --broker-list kafka:9092 

  >hi everybody  
  >i am producer

  ```
- 컨슈머 결과 보기


  ```
  hi everybody  
  i am producer
  ```
- docker-compose 컨테이너 내리기

  ```
  docker-compose down

  ✔ Container learningspoons-kafka-1      Removed                                                                                                                                                        10.3s 
  ✔ Container learningspoons-zookeeper-1  Removed                                                                                                                                                         0.5s 
  ✔ Network learningspoons_default        Removed 

  ```
