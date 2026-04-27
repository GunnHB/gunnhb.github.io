---
title: "[Ticket Reservation System] 성능 최적화"
description: WAS와 Redis를 이용한 성능 최적화 과정
date: 2026-04-27 00:00:00 +09:00
categories: [Textbook, Linux]
tags:
  [
    ComputerScience,
    Textbook,
    Linux
  ]
---

![](/assets/img/post/Textbook/Ubuntu%20Linux/ubuntu.png){: width="350" height="250" }

## 목표
- 대규모 트래픽(티켓 오픈 시점의 동시 접속)을 감당할 수 있는 확장 가능한 백엔드 인프라 아키텍처 설계 및 검증

### 프로젝트 환경
- 하드웨어 환경
  - AMD Ryzen 5 3500U - 8 Threads
  - 8GB RAM

- 테스트 조건
  - Apache Bench 활용
  - 동시 접속자 1,000명, 총 요청 50,000건

- 사용 기술
  - Ubuntu
  - Nginx
  - Docker
  - Uvicorn
  - Apache Bench
  - Redis

## 성능 변화 성능 측정
- 서버의 병목 지점을 파악하고 아키텍쳐 조건을 변경해 가며 하드웨어의 물리적 한계치와 소프트웨어적 개선점 도출

### 1. 최초 부하 테스트
#### 아키텍쳐 조건
- **Nginx**: 기본 설정
- **WAS**: 파이썬 도커 컨테이너 1대 가동 (동기식 처리, DB 조회 0.5초 딜레이 적용)
- **캐시**: 없음

#### 결과
- **RPS**: 1,382
- **최대 지연**: 3.3초 (3,320ms)

#### 분석
- 1,000건의 동시 접속이 발생하자 대기열의 지연 시간이 급격히 증가

<details markdown="1">
<summary>로그</summary>

```
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)


Server Software:        nginx/1.24.0
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /
Document Length:        28 bytes

Concurrency Level:      1000
Time taken for tests:   36.173 seconds
Complete requests:      50000
Failed requests:        0
Total transferred:      9300000 bytes
HTML transferred:       1400000 bytes
Requests per second:    1382.26 [#/sec] (mean)
Time per request:       723.451 [ms] (mean)
Time per request:       0.723 [ms] (mean, across all concurrent requests)
Transfer rate:          251.08 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   5.6      0      58
Processing:     2  715 657.8    652    3320
Waiting:        2  715 657.8    652    3320
Total:          2  716 657.6    653    3320

Percentage of the requests served within a certain time (ms)
  50%    653
  66%    718
  75%    923
  80%   1165
  90%   1628
  95%   2007
  98%   2743
  99%   2871
 100%   3320 (longest request)

```
</details>

### 2. Scale-out 및 로드밸런싱 도입 (최고 성능 달성)
#### 아키텍쳐 조건
- **Nginx**: 성능 튜닝 (`worker_connections 4096`)
- **WAS**: 파이썬 도커 컨테이너 **3대**로 수평 확장 (Round Robin 분산 적용)
- **캐시**: 없음

#### 결과
- **RPS**: 1,450
- **최대 지연**: 2.0초(2,083ms)

#### 분석
- Nginx가 CPU 스레드를 100% 활용하여 앞단 병목을 해소
- 현재 하드웨어 사양에서 뽑아낼 수 있는 물리적인 **최대 처리량 달성**

<details markdown="1">
<summary>로그</summary>

```
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)


Server Software:        nginx/1.24.0
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /
Document Length:        28 bytes

Concurrency Level:      1000
Time taken for tests:   34.463 seconds
Complete requests:      50000
Failed requests:        0
Total transferred:      9300000 bytes
HTML transferred:       1400000 bytes
Requests per second:    1450.85 [#/sec] (mean)
Time per request:       689.250 [ms] (mean)
Time per request:       0.689 [ms] (mean, across all concurrent requests)
Transfer rate:          263.53 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   5.9      0      58
Processing:     2  681 543.2    689    2083
Waiting:        2  681 543.2    689    2083
Total:          2  682 542.9    690    2083

Percentage of the requests served within a certain time (ms)
  50%    690
  66%    849
  75%    988
  80%   1170
  90%   1483
  95%   1762
  98%   1894
  99%   1927
 100%   2083 (longest request)

```
</details>

### 3. Redis 캐시 도입
#### 아키텍쳐 조건
- **Nginx**: 성능 튜닝 유지
- **WAS**: 네트워크 에러 해결을 위해 파이썬 컨테이너 1대만 가동
- **캐시**: Redis 연동 완료 (응답의 99%를 캐시로 반환하도록 설정)

#### 결과
- **RPS**: 1,188
- **최대 지연**: 2.2초(2,225ms)
- **캐시 방어 성공**: 49,650건

#### 분석
- DB 연산을 `Redis`가 0.001초만에 처리
- `Failed request: 49650`
  - DB 조회 응답과 Redis 캐시 응답의 길이가 달라 발생
  - `99%`의 DB 접근을 방어

<details markdown="1">
<summary>로그</summary>

```
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)


Server Software:        nginx/1.24.0
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /
Document Length:        47 bytes

Concurrency Level:      1000
Time taken for tests:   42.054 seconds
Complete requests:      50000
Failed requests:        49650
   (Connect: 0, Receive: 0, Length: 49650, Exceptions: 0)
Total transferred:      10398950 bytes
HTML transferred:       2498950 bytes
Requests per second:    1188.94 [#/sec] (mean)
Time per request:       841.085 [ms] (mean)
Time per request:       0.841 [ms] (mean, across all concurrent requests)
Transfer rate:          241.48 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   6.9      0      78
Processing:     3  820 514.7    819    2225
Waiting:        3  820 514.7    819    2225
Total:          3  821 514.2    819    2225

Percentage of the requests served within a certain time (ms)
  50%    819
  66%    910
  75%   1011
  80%   1155
  90%   1581
  95%   1847
  98%   2034
  99%   2110
 100%   2225 (longest request)

```
</details>

### 4. 동기/비동기 블로킹과 설정 오류의 늪
#### 아키텍쳐 조건
- **Nginx**: 성능 튜닝 유지
- **WAS**: 파이썬 컨테이너 3대 (비동기 async def를 적용했으나, 내부 Redis 무전기는 구형 동기식 라이브러리 사용 / 도커 DNS 매핑 누락)
- **캐시**: Redis 연동 완료

#### 결과
- **RPS**: 216
- **최대 지연**: 24.3초(24,399ms)

#### 분석
- 비동기 함수 내에서 동기식 통신이 발생하여 서버의 메인 스레드가 마비
- DNS 인식 불가 에러로 인해 모든 요청이 88바이트의 에러 메시지만 반환

<details markdown="1">
<summary>로그</summary>

```
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)


Server Software:        nginx/1.24.0
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /
Document Length:        88 bytes

Concurrency Level:      1000
Time taken for tests:   230.840 seconds
Complete requests:      50000
Failed requests:        0
Total transferred:      12300000 bytes
HTML transferred:       4400000 bytes
Requests per second:    216.60 [#/sec] (mean)
Time per request:       4616.793 [ms] (mean)
Time per request:       4.617 [ms] (mean, across all concurrent requests)
Transfer rate:          52.03 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   6.2      0      65
Processing:     5 4457 4918.2   2272   24399
Waiting:        5 4457 4918.2   2272   24399
Total:          5 4458 4917.6   2275   24399

Percentage of the requests served within a certain time (ms)
  50%   2275
  66%   4776
  75%   7107
  80%   8184
  90%  11063
  95%  15370
  98%  18963
  99%  19677
 100%  24399 (longest request)

```
</details>

### 5. 최적화 완료 및 캐시 스탬피드 현상 발견
#### 아키텍쳐 조건
- **Nginx**: 성능 튜닝 유지
- **WAS**: 파이썬 컨테이너 3대 (Host IP 직통 연결)
- **캐시**: 비동기 전용 Redis 라이브러리 연동

#### 결과
- **RPS**:1,240
- **최대 지연**: 2.9초(2,962ms)
- **캐시 방어 성공**: 46,999건

#### 분석
- 최적의 조건으로 수정했음에도 지연 시간이 소폭 상승
- 비동기 WAS 3대가 동시에 열린 상태에서 캐시가 만료되는 찰나, 수천 명의 대기열이 동시에 DB 로직으로 쏟아져 들어가는 **캐시 스템피드**<sup>Cache Stampede</sup> 현상이 발생

<details markdown="1">
<summary>로그</summary>

```
This is ApacheBench, Version 2.3 <$Revision: 1903618 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)


Server Software:        nginx/1.24.0
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /
Document Length:        47 bytes

Concurrency Level:      1000
Time taken for tests:   40.298 seconds
Complete requests:      50000
Failed requests:        46999
   (Connect: 0, Receive: 0, Length: 46999, Exceptions: 0)
Total transferred:      10390997 bytes
HTML transferred:       2490997 bytes
Requests per second:    1240.76 [#/sec] (mean)
Time per request:       805.955 [ms] (mean)
Time per request:       0.806 [ms] (mean, across all concurrent requests)
Transfer rate:          251.81 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   6.9      0      79
Processing:     2  785 712.1    622    2962
Waiting:        2  785 712.1    622    2962
Total:          2  786 711.8    625    2962

Percentage of the requests served within a certain time (ms)
  50%    625
  66%    786
  75%   1184
  80%   1402
  90%   1971
  95%   2219
  98%   2352
  99%   2915
 100%   2962 (longest request)

```
</details>

## 트러블슈팅
- Nginx와 WAS를 비동기 구조로 확장했음에도 응답 지연 시간이 24초까지 치솟는 현상이 발생
- 코드 분석 결과, Redis 통신 라이브러리가 동기식으로 작동하여 **파이썬의 이벤트 루프 자체를 블로킹**하는 것을 확인

**수정 전**
```py
import redis  # 기본 동기식 라이브러리

@app.get("/")
async def read_root():
    cached_data = r.get("baseball_ticket")
```

**수정 후**
```py
import redis.asyncio as redis  # 비동기 전용 라이브러리로 전면 교체

@app.get("/")
async def read_root():
    cached_data = await r.get("baseball_ticket")
```

## 결론
1. 단일 노드의 한계 확인
  - Nginx 튜닝과 서버 수평 확장을 통해 하드웨어가 감당할 수 있는 최대 임계점을 수치로 확인
  - 컨테이너를 무작정 늘린다고 해서 성능이 좋아지는 것이 아닌 문맥 교환 오버헤드로 인해 물리적 한계에 부딪힘을 체감

2. 분산 환경의 부작용 도출
  - 비동기 서버를 다중으로 배치학고 캐시를 적용할 경우 특정 시점에 트래픽이 폭발(캐시 스탬피드)하여 아키텍쳐의 새로운 취약점이 될 수 있음을 직접 확인