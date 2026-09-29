# 1주차 산출물 — Docker Compose 통합 환경

## 구성
- db: PostgreSQL 16, 호스트 포트 5432, pgdata 볼륨에 데이터 저장
- admin: dpage/pgadmin4:9.18, http://localhost:8081
- app: app/Dockerfile로 빌드하는 Python 출력 실습 앱. DB 접속 코드는 없습니다.
- depends_on은 시작 순서를 지정하며 DB의 연결 준비 완료를 보장하지 않습니다.

## 실행
```bash
cd ~/data-eng/week1
cp .env.example .env
nano .env
docker compose config --quiet
docker compose up -d --build
docker compose ps -a
docker compose logs app
```
새로 clone한 경우 저장소 폴더에서 실행하세요. 기존 .env가 있다면 복사로 덮어쓰지 마세요.
.env의 대체 입력값 두 개를 실습용 비밀번호로 바꾸세요. .env는 Git에서 제외됩니다.
기존 pgdata 볼륨의 DB 비밀번호는 환경변수 변경만으로 바뀌지 않습니다. 기존 비밀번호를 유지하거나 DB에서 별도로 변경해야 합니다.

## 접속 정보
- pgAdmin 이메일: admin@example.com
- pgAdmin 비밀번호: .env의 PGADMIN_DEFAULT_PASSWORD
- pgAdmin 서버 등록: Host db, Maintenance database pipeline, Username engineer
- DB 비밀번호: .env의 POSTGRES_PASSWORD
- Compose 네트워크 DB 주소: db:5432; 호스트 PC: localhost:5432

## 확인 및 종료
```bash
docker compose exec db psql -U engineer -d pipeline -c "SELECT current_database();"
docker compose down
docker compose up -d
```
조회 결과는 pipeline입니다. down은 pgdata를 보존하며, 재시작 후 데이터 보존을 확인합니다.
down -v는 볼륨을 삭제하므로 데이터 보존 실습에 사용하지 않습니다.

## 실습 결과
다음은 사용자가 실습 당시 확인한 결과이며 공개용 정리 과정에서 새로 측정한 결과가 아닙니다.
- db와 admin은 실행 상태였습니다.
- app은 아래 내용을 출력하고 Exited (0)으로 정상 종료했습니다.
- current_database() 조회 결과는 pipeline이었습니다.
```text
데이터 파이프라인 앱이 실행되었습니다.
파이썬 버전: 3.12.14
이 줄은 재빌드 실험용입니다.
두 번째 실험
```
python:3.12 및 python:3.12-slim 태그의 패치 버전은 향후 빌드 시 달라질 수 있습니다.

## 겪은 문제와 해결
- compose-test 실습에서 Compose YAML의 착오로 service로 입력해서 에러가 났습니다. GPT 도움을 받아 services:로 수정했습니다.
- docker-compose.yml과 README.md 작성에는 nano 편집기를 사용했습니다.

## 이미지 및 빌드 기록
| 항목 | 확인값 | 조건 |
| --- | --- | --- |
| data-app 이미지 | 195MB | 실습 당시 크기 |
| good-layers 빌드 | 9.3초 | 해당 실습 빌드 시간 |
| Compose 재빌드 | 1.6초 | 캐시 빌드 |

빌드 조건과 캐시 여부가 다르므로 9.3초와 1.6초를 직접 비교하지 않습니다.
data-app과 good-layers는 당시 실습 이름이며 현재 Compose에는 image 이름이 별도로 지정되지 않았습니다.

## 공개 파일
docker-compose.yml, README.md, app/Dockerfile, app/app.py, app/requirements.txt, .env.example, .gitignore

다른 주차 파일, 로그, 백업, .env, 별도 Dockerfile 실험 파일은 제외합니다.
