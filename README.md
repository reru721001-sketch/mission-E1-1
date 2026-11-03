# 개발 워크스테이션 구축 (Terminal · Docker · Git)

## 0) 개요
터미널, Docker, Git을 직접 써 보고 결과를 정리한 저장소입니다.
- 터미널로 파일·폴더를 다루고 권한을 바꿔 봤습니다.
- Docker로 웹 서버를 만들어 브라우저로 접속해 봤습니다.
- 결과물을 Git으로 GitHub에 올렸습니다.

실행 결과는 모두 `logs/` 폴더에 있고, `scripts/` 를 실행하면 똑같이 다시 할 수 있습니다.

## 1) 실행 환경
| 항목 | 값 |
|---|---|
| OS | Windows 11 Pro |
| 쉘 / 터미널 | Git Bash (bash 5.3) |
| Docker | Docker Desktop 4.97.0 (Engine 29.9.2) |
| Git | 2.56.0 |

## 2) 체크리스트 · 검증 방법
| 항목 | 확인한 명령 | 확인한 것 | 결과 |
|---|---|---|---|
| ✅ 터미널 | `mkdir`, `cp`, `mv`, `rm`, `ls` | 파일·폴더 만들기/복사/이동/삭제 | [3)](#3-터미널), [로그](logs/01_terminal_perm.log) |
| ✅ 권한 | `chmod`, `ls -l` | 바꾸기 전/후 비교 | [4)](#4-권한), [로그](logs/02_docker.log) |
| ✅ Docker 설치 | `docker --version`, `docker info` | 설치·실행 여부 | [5)](#5-docker-설치--운영), [로그](logs/02_docker.log) |
| ✅ Docker 운영 | `images`, `ps -a`, `logs`, `stats` | 이미지·컨테이너 상태 | [5)](#5-docker-설치--운영), [로그](logs/02_docker.log) |
| ✅ Dockerfile | `docker build`, `docker run` | 내 페이지가 뜨는지 | [6)](#6-dockerfile-웹-서버), [로그](logs/02_docker.log) |
| ✅ 포트 | `-p 8080:80`, `-p 8081:80` | 브라우저 접속 | [7)](#7-포트-접속) |
| ✅ 바인드 마운트 | `-v 폴더:경로` | 수정 전/후 비교 | [8)](#8-바인드-마운트), [로그](logs/02_docker.log) |
| ✅ 볼륨 | `docker volume` | 삭제 전/후 비교 | [9)](#9-볼륨), [로그](logs/02_docker.log) |
| ✅ Git / GitHub | `git config`, VSCode | 설정·연동 | [10)](#10-git--github) |
| ✅ (보너스) Compose | `docker compose` | 컨테이너 2개 동시 실행 | [11)](#11-보너스-compose), [로그](logs/03_compose.log) |

---

## 3) 터미널
```bash
$ mkdir -p docs backup                  # 폴더 만들기
$ echo "hello codyssey" > note.txt      # 파일 만들기
$ ls -la                                # 목록 보기
drwxr-xr-x  backup
drwxr-xr-x  docs
-rw-r--r--  note.txt
$ cp note.txt backup/note_copy.txt      # 복사
$ mv note.txt docs/renamed.txt          # 이동 + 이름 바꾸기
$ rm backup/note_copy.txt               # 삭제
```
- **절대 경로**: `/` 부터 전부 적은 경로. 어디서든 같은 곳.
- **상대 경로**: 지금 위치 기준 경로(`../`). 위치에 따라 달라짐.

## 4) 권한
Windows에서는 `chmod` 가 적용되지 않아서 우분투 컨테이너 안에서 했습니다.
```bash
$ chmod 755 f.txt; ls -l    -rwxr-xr-x  f.txt   # 644 → 755
$ chmod 700 d; ls -ld d     drwx------  d       # 755 → 700
```
- 숫자 = 읽기(4) + 쓰기(2) + 실행(1), 세 자리는 **나 / 그룹 / 다른 사람**
- `755`: 나는 전부, 남은 읽기·실행 / `644`: 나는 읽기·쓰기, 남은 읽기만

## 5) Docker 설치 · 운영
```bash
$ docker --version
Docker version 29.9.2
$ docker info              # Server 정보가 나오면 실행 중
Server Version: 29.9.2
$ docker run --rm hello-world
Hello from Docker!
$ docker images
hello-world   25.9kB
ubuntu        119MB
$ docker ps -a
my-web-8080   Up (healthy)   0.0.0.0:8080->80/tcp
my-web-8081   Up (healthy)   0.0.0.0:8081->80/tcp
$ docker logs my-web-8080
"GET / HTTP/1.1" 200
$ docker stats --no-stream
my-web-8080   CPU 0.00%   MEM 14.97MiB
```
**attach vs exec** ([로그](logs/04_attach_exec.log))
- `attach`: 컨테이너의 메인 프로그램에 연결 → exit 하면 **컨테이너 꺼짐**
- `exec`: 새 프로그램을 하나 더 실행 → exit 해도 **컨테이너 계속 실행**

## 6) Dockerfile 웹 서버
소스: [app/site/index.html](app/site/index.html) · [app/Dockerfile](app/Dockerfile)

nginx 이미지에 내 홈페이지를 넣어서 새 이미지를 만들었습니다.
```bash
$ docker build -t my-web:1.0 ./app
naming to docker.io/library/my-web:1.0 done
$ docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
$ curl http://localhost:8080
<h1>Hello from my custom NGINX!</h1>
```

## 7) 포트 접속
`-p 8080:80` = 내 PC 8080번으로 접속하면 컨테이너 80번으로 연결

![8080 접속](docs/screenshots/port-8080.png)
![8081 접속](docs/screenshots/port-8081.png)

## 8) 바인드 마운트
내 PC 파일을 고치면 컨테이너에도 바로 반영됩니다.
```bash
$ docker run -d -p 8082:80 -v "$PWD/bind-site":/usr/share/nginx/html nginx:1.27-alpine
$ curl localhost:8082      # 수정 전
version: v1
$ sed -i "s/v1/v2/" bind-site/index.html     # 내 PC에서 수정
$ curl localhost:8082      # 수정 후
version: v2
```

## 9) 볼륨
컨테이너를 지워도 볼륨에 저장한 데이터는 남습니다.
```bash
$ docker volume create mydata
$ docker run -d --name vol-test -v mydata:/data ubuntu sleep infinity
$ docker exec vol-test sh -c "echo hi > /data/hello.txt"
$ docker rm -f vol-test                      # 컨테이너 삭제
$ docker run -d --name vol-test2 -v mydata:/data ubuntu sleep infinity
$ docker exec vol-test2 cat /data/hello.txt  # 새 컨테이너에서 확인
hi
```

## 10) Git · GitHub
```bash
$ git config --global user.name "손건"
$ git config --global user.email "****@icloud.com"
$ git config --global init.defaultBranch main
$ git config --list
user.name=손건
user.email=****@icloud.com
init.defaultbranch=main
```
VSCode에서 GitHub 로그인 후 저장소와 연결했습니다.

![VSCode GitHub 연동](docs/screenshots/vscode-github.webp)

## 11) 보너스: Compose
[docker-compose.yml](docker-compose.yml) 파일 하나로 웹 서버 + Redis를 같이 켰습니다.
```bash
$ docker compose up -d
$ docker compose ps
web     Up   0.0.0.0:8090->80/tcp
cache   Up   6379/tcp
$ docker compose down
```

---

## 12) 트러블슈팅

**#1 Windows에서 `chmod` 가 안 됨**
- 문제: 권한을 바꿔도 그대로였다.
- 원인 가설: Windows 파일 시스템은 리눅스 권한을 지원 안 하는 것 같다.
- 확인: 우분투 컨테이너에서 하니 잘 바뀌었다.
- 해결: 권한 실습은 컨테이너 안에서 했다.

**#2 Docker Desktop이 안 켜짐**
- 문제: "Virtual Machine Platform is not enabled" 오류.
- 원인 가설: 필요한 Windows 기능이 아직 적용 안 된 것 같다.
- 확인: 기능은 켰지만 재부팅을 안 한 상태였다.
- 해결: 재부팅 후 정상 실행.

**#3 로그 파일 경로 오류**
- 문제: `tee: No such file or directory`
- 원인 가설: `cd` 로 폴더를 옮겨서 상대 경로가 틀어진 것 같다.
- 확인: `pwd` 로 보니 다른 폴더에 있었다.
- 해결: 로그 경로를 절대 경로로 바꿨다.

## 13) 보안
- `.gitignore` 로 비밀 파일(`.env`, `*.key`)이 올라가지 않게 했습니다.
- 이메일 등 개인 정보는 가렸습니다.
