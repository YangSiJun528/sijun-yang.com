# 도커로 PostgreSQL 띄우기 (로컬 개발용)

> 주의: 로컬에서 테스트하거나 개발용으로만 사용하기. 배포용으로 쓴 글 아님

## 1. PostgreSQL 컨테이너 생성

```bash
docker run --name my-postgres \
    -e POSTGRES_PASSWORD=<postgres계정비번> \
    -e PGDATA=/var/lib/postgresql/data/pgdata \
    -v /Users/<PC사용자이름>/Dev/docker/postgres:/var/lib/postgresql/data \
    -p 5432:5432 \
    -d postgres
```

* `POSTGRES_PASSWORD` : 기본 `postgres` 계정 비밀번호
* `PGDATA` : PostgreSQL 데이터 디렉터리 지정
* `-v` : 로컬 경로를 컨테이너 데이터 디렉터리와 연결 (영속성 확보) - 원하는 경로로 커스텀 가능
* `-p` : 호스트 포트 → 컨테이너 포트 매핑

## 2. 컨테이너 내부 접속

```bash
docker exec -it my-postgres bash
```

- 컨테이너 내부에서 쉘로 접속 - docker desktop 사용 시 exec 탭에서 바로 실행 가능  

```bash
psql -U postgres
```

## 3. 데이터베이스 및 사용자 생성

```sql
-- 데이터베이스 생성
CREATE DATABASE <디비이름>;

-- DB 전용 사용자 생성
CREATE ROLE <디비관리자이름> WITH ENCRYPTED PASSWORD '<디비관리자비밀번호>';

-- 권한 부여
GRANT ALL ON DATABASE <디비이름> TO <디비관리자이름>;

-- 소유자 변경
ALTER DATABASE <디비이름> OWNER TO <디비관리자이름>;
```

> 서비스별로 DB/사용자를 분리해서 서로 격리함

---

## 스프링에서 연결

```properties
spring.sql.init.platform=postgresql
spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/<디비이름>
spring.datasource.username=<디비관리자이름>
spring.datasource.password=<디비관리자비밀번호>
spring.datasource.driver-class-name=org.postgresql.Driver
```

DB 관리 툴에서도 같은 환경변수 사용해서으로 접속 가능
