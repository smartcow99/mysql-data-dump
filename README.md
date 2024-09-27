# mysql-data-dump

## 개요

~ 상황 발생
귀찮거나 문제 상황임
이를 해결하기 위해 데이터베이스 덤프를 사용해야함
그러니까 나머지는 곽병찬이 정리해!

### 1. 특정 데이터베이스 `fisa` Dump

다음 명령어를 사용하여 `fisa` 데이터베이스만 dump 파일로 저장할 수 있습니다.

```bash
docker exec mysqldb mysqldump -u root -p[your_password] fisa > /path/to/backup/fisa_dump.sql
```

이 명령어는 `mysqldb` 컨테이너에서 `fisa` 데이터베이스의 데이터를 dump하여 `fisa_dump.sql` 파일로 저장합니다.

### 2. 새로운 `newmysqldb` 컨테이너 생성 후 Dump 파일 반영

### A. **`docker-compose.yml` 사용**

새로운 컨테이너에 특정 데이터베이스(`fisa`)를 복원하는 `docker-compose.yml` 파일을 작성할 수 있습니다.

### 1. `docker-compose.yml` 파일 예시

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: newmysqldb
    environment:
      MYSQL_ROOT_PASSWORD: your_password
      MYSQL_DATABASE: fisa
    volumes:
      - /path/to/backup:/docker-entrypoint-initdb.d
    ports:
      - "3306:3306"
    restart: always
```

- `MYSQL_DATABASE`를 `fisa`로 설정하여 기본적으로 `fisa` 데이터베이스를 생성하도록 합니다.
- `/path/to/backup/fisa_dump.sql` 경로에 있는 dump 파일을 복구할 수 있도록 설정합니다.

### 2. `docker-compose` 명령어 실행

```bash
docker-compose up -d
```

이 명령어는 `fisa_dump.sql` 파일을 `newmysqldb` 컨테이너에 복구하면서 컨테이너를 생성합니다.

---

### B. **`crontab`을 사용한 자동화**

`crontab`을 이용해 특정 데이터베이스를 주기적으로 dump하고 복구하는 작업을 자동화할 수 있습니다.

### 1. 스크립트 작성

`fisa_backup_restore.sh`라는 스크립트를 작성합니다.

```bash
#!/bin/bash

# 특정 데이터베이스 fisa dump
docker exec mysqldb mysqldump -u root -p[your_password] fisa > /path/to/backup/fisa_dump.sql

# Remove old newmysqldb container if exists
docker rm -f newmysqldb

# Create newmysqldb container
docker run --name newmysqldb -e MYSQL_ROOT_PASSWORD=your_password -d -p 3306:3306 mysql:8.0

# Restore fisa database into the newmysqldb container
docker exec -i newmysqldb mysql -u root -p[your_password] fisa < /path/to/backup/fisa_dump.sql
```

이 스크립트는:

1. `fisa` 데이터베이스를 dump하고,
2. 기존 `newmysqldb` 컨테이너를 제거한 후,
3. 새로운 `newmysqldb` 컨테이너를 생성하고,
4. `fisa_dump.sql` 파일을 복구합니다.

### 2. Crontab 설정

주기적으로 이 작업을 실행하도록 `crontab`에 등록합니다.

```bash
crontab -e
```

다음과 같은 줄을 추가하여 매일 오전 2시에 실행되도록 설정합니다.

```bash
0 2 * * * /path/to/fisa_backup_restore.sh >> /path/to/log/fisa_backup_restore.log 2>&1

```

---

### 요약

특정 데이터베이스(`fisa`만)를 dump하여 새 컨테이너에 복원하는 작업은 `mysqldump` 명령어로 간단히 가능하며, `docker-compose` 또는 `crontab`을 사용하여 이를 자동화할 수 있습니다.


docker exec mysqldb mysqldump -u root -p[root] --all-databases > ./backup/mysqldb_dump.sql
![image](https://github.com/user-attachments/assets/8c151399-5b1b-4751-9958-516792679b8b)
