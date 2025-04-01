1. Eureka에 등록된 Auth IP를 Gateway가 호출 실패함
- **원인**: Fargate 태스크 컨테이너가 자체적으로 사용하는 내부 네트워크 IP가 등록됨.
- **해결**: 태스크 각각의 퍼블릭 IP가 아니라 하나의 EndPoint로 접근할 수 있도록 로드 밸런서 설정
<br>

2. ECS 태스크 Unhealthy
- **원인**: 로드 밸런서 상태 검사 경로가 `/` 였음. Actuator는 기본적으로 `/actuator/health` 제공
- **해결**: EC2 > 대상 그룹 > 해당 대상 그룹 체크 > 상단 작업 탭에서 상태 검사 설정 편집 > 상태 검사 경로 `/actuator/health`로 수정
<br>

3. `Too many connections` – RDS MySQL 연결 초과

```groovy
java.sql.SQLNonTransientConnectionException: Too many connections
```

- auth 태스크 3개에서 1개로 줄이고 Config의 authentication.yml에 maximum-pool-size 설정 추가
    
    ```groovy
    hikari:
    	...기타설정
    	maximum-pool-size: 10      # ✅ 커넥션 풀 최대 수
    	minimum-idle: 5            # 최소 유지 커넥션 수
    	idle-timeout: 30000        # 유휴 커넥션 최대 유지 시간(ms)
    	max-lifetime: 1800000      # 커넥션 최대 생존 시간(ms)
    	connection-timeout: 30000  # 커넥션 가져올 때까지 최대 대기 시간(ms)
    ```
<br>
    

4. `Eureka` 등록 실패 (`Connection refused`)

```groovy
I/O error on POST request for "http://localhost:8761/eureka/apps/BOARD": Connect to http://localhost:8761 [localhost/127.0.0.1] failed: Connection refused
```

- Config `board.yml` 유레카 설정부분 프로퍼티명 표기법 수정(케밥 케이스 → 카멜 케이스)
    
    ```groovy
    eureka:
      ...기타설정
      client:
        ... 기타설정
        serviceUrl:
          defaultZone: http://54.215.85.146:8761/eureka/
    ```
