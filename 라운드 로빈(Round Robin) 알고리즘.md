# 라운드 로빈(Round Robin) 알고리즘

라운드 로빈은 프로세스 스케줄링, 네트워크 패킷 처리, 부하 분산 등 다양한 컴퓨터 시스템에서 사용되는 기본적인 스케줄링 알고리즘입니다.

## 기본 개념

- 모든 프로세스나 작업에 **동일한 우선순위**를 부여하고 **순환식**으로 자원을 할당
- 각 프로세스에 **일정한 시간 할당량(time quantum 또는 time slice)** 부여
- 할당된 시간이 끝나면 프로세스는 **강제로 중단**되고 **대기 큐의 맨 뒤**로 이동
- 다음 순서의 프로세스가 CPU를 할당받음

## 작동 방식

1. 모든 프로세스가 Ready Queue에 대기
2. 스케줄러가 큐의 맨 앞 프로세스에 CPU 할당
3. 프로세스는 할당된 시간 동안만 실행
4. 시간이 다 되면 프로세스 선점(preemption)하고 큐의 맨 뒤로 이동
5. 1-4 과정을 반복

## 특징

- **공정성**: 모든 프로세스에 동등한 CPU 시간 제공
- **응답성**: 긴 작업이 CPU를 독점하지 못하게 하여 짧은 작업의 응답 시간 개선
- **구현 용이성**: 비교적 간단한 구현
- **선점형(Preemptive)**: 프로세스가 완료되지 않아도 정해진 시간 후 강제 중단

## 성능 영향 요소

- **시간 할당량(Time quantum)**:
    - 너무 작으면: 문맥 교환(context switching) 오버헤드 증가
    - 너무 크면: FCFS(First-Come, First-Served)와 유사해짐
    - 일반적으로 10-100ms 범위 사용

## 응용 분야

1. **CPU 스케줄링**: 운영체제의 프로세스 스케줄링
2. **네트워크 패킷 처리**: 패킷 스케줄링(특히 QoS 구현)
3. **로드 밸런싱**: 웹 서버 등의 부하 분산
4. **실시간 시스템**: 소프트 실시간 시스템의 태스크 스케줄링

## 장단점

### 장점
- 모든 프로세스가 공정하게 실행됨
- 응답 시간 예측 가능
- 한 프로세스의 독점 방지

### 단점
- 시간 할당량 설정에 따라 성능 크게 좌우
- 우선순위가 필요한 작업에는 부적합
- 문맥 교환 오버헤드 발생

라운드 로빈은 단순하면서도 효과적인 알고리즘으로, 특히 시분할(time-sharing) 시스템에서 널리 사용되며 다양한 고급 스케줄링 알고리즘의 기초가 되었습니다.
