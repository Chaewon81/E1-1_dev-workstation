# E1-1 개발 워크스테이션 과제

개발 작업에 필요한 터미널, Docker, Git/GitHub 환경을 준비하고, 명령과 결과를 재현할 수 있도록 정리한 과제입니다.

- GitHub 저장소: [E1-1_dev-workstation](https://github.com/Chaewon81/E1-1_dev-workstation)
- 웹 페이지: 학번과 이름을 포함한 간단한 NGINX 정적 페이지
- 보너스: Docker Compose 멀티 서비스 실행, 환경 변수, SSH 인증

## 실행 환경

| 항목 | 버전 / 환경 |
|---|---|
| 운영체제 | Windows, WSL 2 (Ubuntu 26.04.1 LTS) |
| 터미널 | Windows PowerShell, Ubuntu Bash |
| Docker Desktop | 4.97.0 |
| Docker Engine | 29.9.2 |
| Docker Compose | v5.6.1 |
| Git | 2.56.0.windows.4 |
| VS Code | GitHub 계정 로그인 및 로컬 저장소 확인 |

## 디렉터리 구조

~~~text
dev-workstation-assignment/
├── .gitignore
├── Dockerfile
├── README.md
├── bonus/
│   └── compose.yaml
├── cli-practice/
│   ├── hello.txt
│   ├── permission-test.txt
│   ├── permission-dir/
│   │   └── .gitkeep
│   └── archive/  # CLI 이동·삭제 실습에서 사용
├── screenshots/
├── site/
│   └── index.html
~~~

## 수행 체크리스트

- [x] CLI에서 현재 경로 확인, 파일·폴더 생성, 내용 읽기, 복사, 이동, 삭제
- [x] Ubuntu Linux에서 파일·디렉터리 권한을 chmod로 변경
- [x] Docker 설치 및 엔진 상태 확인
- [x] hello-world, Ubuntu 컨테이너 실행 및 상태·로그·이미지·리소스 확인
- [x] Dockerfile로 NGINX 이미지 빌드 및 포트 매핑 웹 페이지 확인
- [x] 바인드 마운트와 볼륨 데이터 영속성 확인
- [x] Git 설정, 로컬 저장소, GitHub 원격 연결, VS Code 연동 확인
- [x] Compose로 두 서비스 실행, 상태·로그·환경 변수·서비스 통신 확인
- [x] SSH 공개키 등록 및 GitHub SSH 인증 확인

## CLI 실습

PowerShell에서 cli-practice 폴더 안의 파일 조작을 연습했습니다.

~~~powershell
Set-Content source.txt "CLI file operation practice"
Copy-Item source.txt copied.txt
Get-Content copied.txt
New-Item -ItemType Directory archive
Move-Item copied.txt archive\copied.txt
Remove-Item source.txt
Remove-Item archive\copied.txt
Test-Path source.txt
Test-Path archive\copied.txt
~~~

Get-Content는 파일 내용을 읽고, Test-Path가 삭제 후 False를 출력해 파일이 사라진 것을 확인했습니다. PowerShell에서 파일 조작을 했고, Linux 권한 실습은 Ubuntu Bash에서 별도로 진행했습니다.

~~~bash
ls -ld permission-test.txt permission-dir
chmod 600 permission-test.txt
chmod 700 permission-dir
ls -ld permission-test.txt permission-dir
~~~

변경 전 파일 권한은 644 (rw-r--r--), 디렉터리 권한은 755 (rwxr-xr-x)였습니다. 변경 후 각각 600 (rw-------)과 700 (rwx------)이 됐습니다. 실제 Linux 권한을 확인한 뒤 제출 구조에 필요한 파일을 프로젝트 폴더로 복사했습니다.

## Docker 기본 실습

~~~powershell
docker --version
docker info
docker run hello-world
docker run -it --name ubuntu-practice ubuntu bash
docker images
docker ps
docker ps -a
docker logs ubuntu-practice
docker run -d --name stats-practice ubuntu sleep 300
docker stats --no-stream stats-practice
docker stop stats-practice
~~~

hello-world에서 Hello from Docker!가 출력됐습니다. Ubuntu 컨테이너에서는 pwd, ls -la, echo를 실행했습니다. docker ps는 실행 중인 컨테이너를, docker ps -a는 종료된 컨테이너까지 보여줍니다. docker stats로 실행 중인 컨테이너의 CPU·메모리 사용량도 확인했습니다.

## Dockerfile 웹 서버

Dockerfile은 nginx:alpine을 바탕으로 과제 페이지를 이미지에 복사합니다.

~~~dockerfile
FROM nginx:alpine
COPY site/index.html /usr/share/nginx/html/index.html
EXPOSE 80
~~~

프로젝트 루트에서 빌드하고 컨테이너 포트 80을 호스트 포트 8080에 연결합니다.

~~~powershell
docker build -t dev-workstation-web .
docker run -d -p 8080:80 --name dev-workstation-container dev-workstation-web
~~~

브라우저에서 http://localhost:8080을 열어 페이지를 확인했습니다. 페이지의 학번과 이름은 과제 페이지에 표시한 값입니다.

## 바인드 마운트와 볼륨

바인드 마운트는 Windows 프로젝트 폴더를 컨테이너의 /usr/share/nginx/html에 연결했습니다. docker inspect에서 마운트 유형이 bind이고 읽기 전용(RW=false)인 것을 확인했습니다. 호스트 파일을 컨테이너에서 직접 사용하는 방식입니다.

볼륨은 컨테이너 생명주기와 데이터 생명주기가 다를 수 있음을 확인했습니다.

~~~powershell
docker volume create assignment-data
docker run --name volume-check-1 -v assignment-data:/data ubuntu bash -c "echo persisted-data > /data/check.txt && cat /data/check.txt"
docker rm volume-check-1
docker run --rm -v assignment-data:/data ubuntu cat /data/check.txt
~~~

첫 컨테이너를 삭제한 뒤 새 컨테이너에서 persisted-data가 다시 출력되어 볼륨 데이터가 남는 것을 확인했습니다.

## Git, GitHub, VS Code

- Git 전역 사용자 이름·이메일이 설정되어 있음을 확인했습니다. 이메일 값은 캡처에서 숨겼습니다.
- 기본 브랜치를 main으로 설정하고 프로젝트를 로컬 Git 저장소로 초기화했습니다.
- .gitignore에서 .env, .pem, .key 파일을 제외하도록 했습니다.
- GitHub 원격 저장소 origin을 SSH 주소로 설정하고 git ls-remote origin으로 접속을 확인했습니다.
- VS Code에서 과제 폴더의 Source Control과 GitHub 계정 로그인을 확인했습니다.

~~~powershell
git status --short --branch
git remote -v
git ls-remote origin
git push -u origin main
~~~

## 보너스: Docker Compose

bonus/compose.yaml은 web과 helper 두 서비스를 정의합니다. Compose는 설정 파일 하나로 여러 컨테이너와 네트워크를 함께 관리하므로 같은 구성을 다시 실행하고 공유하기 쉽습니다. Compose가 기본 네트워크를 만들고, 서비스는 이름으로 서로 찾습니다.

~~~powershell
docker compose -f .\bonus\compose.yaml config
docker compose -f .\bonus\compose.yaml up -d
docker compose -f .\bonus\compose.yaml ps
docker compose -f .\bonus\compose.yaml logs --tail 20 web
$env:APP_MODE = "compose-demo"
docker compose -f .\bonus\compose.yaml up -d --force-recreate helper
docker compose -f .\bonus\compose.yaml exec helper sh -c 'echo "APP_MODE=$APP_MODE"; wget -qO- http://web | head -n 5'
Remove-Item Env:APP_MODE
docker compose -f .\bonus\compose.yaml down
~~~

호스트 포트 8082에서 NGINX 페이지를 확인했고, Compose 로그에 GET / 응답 코드 200이 기록됐습니다. APP_MODE의 기본값 bonus와 PowerShell 환경 변수로 덮어쓴 compose-demo를 컨테이너 안에서 확인했습니다. down으로 두 컨테이너와 Compose 네트워크를 정리했습니다.

## 보너스: GitHub SSH

HTTPS와 SSH는 원격 저장소 접근 방식입니다. HTTPS는 Git Credential Manager 또는 토큰으로 인증할 수 있고, SSH는 컴퓨터의 개인키로 인증합니다. GitHub에는 공개키만 등록하고 개인키는 컴퓨터에 보관합니다.

~~~powershell
ssh -T git@github.com
git remote -v
git ls-remote origin
git remote set-url origin git@github.com:Chaewon81/E1-1_dev-workstation.git
~~~

You've successfully authenticated, but GitHub does not provide shell access. 메시지로 SSH 인증을 확인했습니다. SSH 인증 성공은 GitHub 계정 확인이며, 대화형 셸을 여는 기능은 아닙니다.

## 증거 캡처

| 항목 | 증거 |
|---|---|
| 프로젝트 구조 및 CLI 경로·파일 생성 | [01-project-structure.png](screenshots/01-project-structure.png), [02-cli-practice-path.png](screenshots/02-cli-practice-path.png), [03-cli-create-file.png](screenshots/03-cli-create-file.png) |
| Windows 속성 및 Linux 권한 | [04-cli-permission-before-after.png](screenshots/04-cli-permission-before-after.png), [04-linux-chmod.png](screenshots/04-linux-chmod.png) |
| Docker 설치, 컨테이너, 웹, 빌드 | [05-docker-version-info.png](screenshots/05-docker-version-info.png), [06-docker-hello-ubuntu.png](screenshots/06-docker-hello-ubuntu.png), [06-docker-hello-ubuntu-2.png](screenshots/06-docker-hello-ubuntu-2.png), [07-docker-build-run-port.png](screenshots/07-docker-build-run-port.png), [15-docker-build-output.png](screenshots/15-docker-build-output.png) |
| 바인드 마운트 및 볼륨 영속성 | [08-bind-mount-or-volume.png](screenshots/08-bind-mount-or-volume.png), [08-volume-persistence2.png](screenshots/08-volume-persistence2.png) |
| Git 설정, 원격, CLI 파일 조작, 푸시 | [09-git-config-github.png](screenshots/09-git-config-github.png), [10-cli-file-move.png](screenshots/10-cli-file-move.png), [11-cli-file-delete.png](screenshots/11-cli-file-delete.png), [12-github-remote.png](screenshots/12-github-remote.png), [21-github-push.png](screenshots/21-github-push.png) |
| VS Code, Compose, SSH | [14-vscode-github-account.png](screenshots/14-vscode-github-account.png), [16-compose-bonus.png](screenshots/16-compose-bonus.png), [17-compose-browser.png](screenshots/17-compose-browser.png), [18-compose-env-network.png](screenshots/18-compose-env-network.png), [19-compose-down.png](screenshots/19-compose-down.png), [20-github-ssh-test.png](screenshots/20-github-ssh-test.png) |

## 트러블슈팅

### 1. docker 명령을 찾지 못함

PowerShell에서 docker가 명령으로 인식되지 않았습니다. Windows용 Docker Desktop을 설치하고 WSL 2 백엔드를 실행한 뒤 새 PowerShell에서 docker --version과 docker info로 클라이언트와 엔진 연결을 확인했습니다.

### 2. WSL에서 프로젝트 D: 드라이브를 찾지 못함

Ubuntu에서 /mnt/d가 없어 Windows 프로젝트 경로에 접근할 수 없었습니다. WSL 파일을 Windows PowerShell에서 \\wsl.localhost\Ubuntu\home\leech\... 경로로 복사해 해결했습니다. Windows 파일 시스템의 권한은 Linux 파일 시스템과 다르게 처리될 수 있어 chmod 실습은 Ubuntu 홈 디렉터리에서 진행했습니다.

## 보안 및 개인정보

- 토큰, 계정 비밀번호, SSH passphrase, SSH 개인키는 저장소에 포함하지 않았습니다.
- .gitignore에서 .env, .pem, .key를 제외합니다.
- site/index.html의 학번과 이름은 공개해도 된다고 확인한 과제 페이지 값입니다.
