---
title: "Docker 활용하여 리눅스 서버에 파이썬 환경 만들기(주피터 서버)"
author: Seungjoo Ra
date: 2022-10-31 14:00:00 +0900
categories: [Docker, Ubuntu, Python]
tags: [Docker, Jupyter]
math: true
mermaid: False
image:
  path: blog_img/2022_10_31_Docker_jupyter/main.png
  width: 800
  height: 350
  # alt: Docker Jupyter
---

---
**Contents**

{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}
---


# Docker를 사용하는 이유
리눅스 서버에 파이썬 환경을 구성하기 위해서는 프로젝트 관련 파이썬 환경과 패키지 관리를 해줘야 하는데 매번 환경을 맞추기가 쉽지 않습니다. 왜냐하면 패키지 의존성에 의한 에러나 파이썬 버전이 달라 변경된 함수가 있거나 환경변수 설정 등 내가 공들여 해놓은 세팅을 서버를 옮기거나 다른 서버 혹은 나의 환경에 적용할 때마다 OS에 따라 다르게 설정해줘야 하는 번거로움을 없애고 Docker를 활용하여 환경에 맞는 Docker image를 만들어 쉽고 빠르게 환경 구성을 할 수 있기 떄문입니다.
`Dockerfile을 생성해서 하는 방법도 있지만 직접 도커 컨테이너에 접속하여 환경을 구성하였습니다.`
Dockerfile을 만드는 방법은 구글 검색을 해보면 쉽게 설명되어 있는 곳도 많고 <https://docs.docker.com/engine/reference/builder/> 를 참조하면 쉽게 만들 수 있기 때문에 선택하여 하시길 바랍니다.

# 1. Docker 이미지 만들기

### 📌Python Version = 3.8.13

## 1-1. Docker 이미지 Docker 허브에서 가져오기
```bash
docker pull python:3.8

3.8: Pulling from library/python
Digest: sha256:8b0fc420ec9b030c8c788ecd6dc9ad6b2f695dce183dc6a470f74c732e019a4a
Status: Downloaded newer image for python:3.8
docker.io/library/python:3.8

docker images

REPOSITORY       TAG       IMAGE ID       CREATED        SIZE
remote_jupyter   latest    cd58f260bfad   46 hours ago   1.23GB
python           3.8       0a91fd9cc482   3 days ago     912MB
```
`docker images`를 통해 python 3.8 version이 Docker hub에서 Pull된 것을 확인할 수 있습니다.

## 1-2. Docker 컨테이너 생성

```bash
docker run -d -it --name my_python -p 10880:8888 python:3.8

b65eb318f92cbcac4741d0f8a47b526f98ee152c519dff1d260f31cee03d5de1

docker ps -a

CONTAINER ID   IMAGE        COMMAND     CREATED         STATUS         PORTS                     NAMES
b65eb318f92c   python:3.8   "python3"   6 seconds ago   Up 3 seconds   0.0.0.0:10880->8888/tcp   my_python
```
- **옵션 설명**
  * -d : Detached(독립적인) 모드로 docker 내부에 실행되는 요소를 실행하지 않고 백그라운드에서 실행될 수 있도록 함
  * -i : 컨테이너와 연결되어 있지 않아도 계속적인 커맨드 입력을 받음
  * -t : TTY 모드를 사용, Bash를 사용하려면 이 옵션을 설정 필수
  * -p : myport:container port로 포트포워딩

```bash
docker ps -a

CONTAINER ID   IMAGE        COMMAND     CREATED         STATUS         PORTS                     NAMES
b65eb318f92c   python:3.8   "python3"   6 seconds ago   Up 3 seconds   0.0.0.0:10880->8888/tcp   my_python
```
`docker ps -a` 명령을 통해 컨테이너가 잘 실행되고 있는 것을 확인할 수 있습니다.

## 1-3. Docker 컨테이너 CLI 접속

```bash
docker exec -it my_python bin/bash

root@b65eb318f92c:/#
```
docker exec 명령을 사용하고 `-it(cli 접속)` 옵션을 사용하고 \<컨테이너\>에 있는 bin 파일 내에 있는 bash를 실행하겠다는 의미입니다.

## 1-4. Python 환경 구성하기

### 1-4-1. Python 버전 확인
```bash
python --version

Python 3.8.13
```

### 1-4-2. Python 가상환경 생성
vertualenv를 사용하여 환경을 구성하였습니다.

```bash
cd home
mkdir work
pip install virtualenv
```
home 디렉토리로 이동하여 work라는 작업할 폴더를 생성해주었고 home에 가상환경을 생성하기 위해 vertualenv를 다운로드합니다.
```bash
python -m venv .venv
ls -a
.  ..  .venv  work
```
 .venv라는 가상환경 관리 폴더를 생성해줍니다.
 
 ```bash
source ./.venv/bin/activate
which python
/home/.venv/bin/python
 ```
 source 명령을 통해 \<가상환경폴더\>/\<bin\>/activate 내용을 실행
 python 위치를 확인해보면 가상환경 파이썬으로 적용된 것을 확인할 수 있습니다.
 
 ### 1-4-3. 필요 패키지 다운로드
 
 여기서는 쥬피터 서버연동만 기록할 것이기 때문에 필요한 분들은 각자 설치하시기 바랍니다.
 ```bash
 pip install --upgrade pip
 pip install jupyterlab
 ```
 혹시모를 버전 충돌을 방지하기 위해서 pip 버전을 최신으로 업그레이드 해줌
 제 경우는 jupyter lab을 선호하기 때문에 jupyter lab으로 진행했으나 jupyter notebook이나 다른 경우도 과정은 동일하기 때문에 선택하여 사용하시기 바랍니다.

### 1-4-4. apt upgrade 및 vim editor 다운로드
```bash
apt-get update
apt-get install vim
```


### 1-4-4. jupyter 설정파일 생성

```bash
jupyter notebook --generate-config -y

Writing default config to: /root/.jupyter/jupyter_notebook_config.py
```
생성된 경로를 확인

### 1-4-5. jupyter 설정파일 수정
```python
# ipython
Python 3.8.13 (default, Mar 31 2022, 00:26:41)
Type 'copyright', 'credits' or 'license' for more information
IPython 8.2.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from notebook.auth import passwd

In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'argon2:$argon2id$v=19$m=10240,t=10,p=8$dDRJ6dknUMy1277hlE60mg$0aQlg75qiRXJppnYTo6rKVXApyixaYVZH++Rc8SMdp4'
```
ipython을 실행하여 hash 암호를 생성하여 복사합니다.
`ctrl+z`로 ipython을 나옵니다.
```bash
vi ~/.jupyter/jupyter_notebook_config.py
```
vi 에디터로 실행해 줍니다.
]]를 입력하여 가장 맨 밑줄로 이동합니다.
i를 통해 입력 모드로 진입합니다.

```bash
c=get_config()

c.NotebookApp.ip='localhost'
c.NotebookApp.open_browser=False
c.NotebookApp.password='argon2:$argon2id$v=19$m=10240,t=10,p=8$j0lA7sC6xTk3x4b3rd7vFw$DdCUnYmzWUWrfsI4UoV3pA'
c.NotebookApp.password_required=True
c.NotebookApp.port=8888
c.NotebookApp.iopub_data_rate_limit=1.0e10  
c.NotebookApp.terminado_settings={'shell_command': ['/bin/bash']}  # terminal을 bash로 실행
```
가장 밑줄에 내용을 추가해줍니다.

![](blog_img/2022_10_31_Docker_jupyter/note_book_pw_set.png){: width="970", height="159" .shadow}<br>

esc -> :wq 를 입력하여 쓰고 vim 에디터를 종료합니다.

## 1-5. 쥬피터 접속 확인

### 1-5-1. 쥬피터 서버 실행
제 경우 백그라운드에서 쥬피터 서버가 돌면서 cli작업도 하고 싶었기 때문에 쥬피터를 백그라운드 모드로 실행했습니다.
만약 기존에 쥬피터 노트북 실행하던 것처럼 하고자 한다면 `nohup &`을 빼주시기 바랍니다.
```bash
nohup jupyter lab --ip 0.0.0.0 --allow-root &
[1] 880
```
jupyter lab을 `localhost`로 root 권한을 부여하여 실행해줍니다.
PID 880으로 실행된 것을 알 수 있습니다.
`nohup로 실행할 때는 엔터를 두번 쳐 주시면 됩니다.`

```bash
ps -ef

UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 06:39 pts/0    00:00:00 python3
root        16     0  0 06:52 pts/1    00:00:00 bash
root       865    16  0 07:28 pts/1    00:00:00 /home/.venv/bin/python /home/.venv/bin/ipython
root       869     0  0 07:30 pts/2    00:00:00 bin/bash
root       880   869  0 07:38 pts/2    00:00:00 /home/.venv/bin/python /home/.venv/bin/jupyter-lab --ip 0.0.0.0 --allow-
root       882   869  0 07:40 pts/2    00:00:00 ps -ef
```
`ps -ef`로 한번 더 확인을 해줍니다.


### 1-5-2. 쥬피터 접속

1. 웹 브라우저를 실행하여 `127.0.0.1:port(10880)` 또는 `localhost:port(10880)`으로 접속해줍니다.
2. 아까 설정해두었던 패스워드를 입력합니다. 
![](blog_img/2022_10_31_Docker_jupyter/jupyter_login_1.png){: .shadow}

3. /home 디렉토리 기준으로 실행된 jupyter lab을 확인할 수 있습니다.
![](blog_img/2022_10_31_Docker_jupyter/jupyter_lab_page.png){: .normal}


## 1-6. 이미지 파일 save 하기

이제 잘 되는 것을 확인했으니 이미지파일을 로컬파일로 save 해주겠습니다.
리눅스, 우분투, 맥 등 다른 os에서도 Docker만 설치되어 있고 save 한 이미지 파일만 있다면 지금까지 우리가 설정한 환경으로 실행할 수 있습니다.
지금까지 작업했던 컨테이너 내부 cli 이외에 os위에 있는 cli에서 진행합니다.
```bash
docker commit my_python my_python:3.8.13
```
docker commit \<컨테이너 명\> \<커밋할 이미지명:tag\> 로 컨테이너 내용을 이미지로 생성합니다.

```bash
docker save -o my_python.tar my_python:3.8.13
```

docker save -o(파일출력) \<출력할 파일명\> \<이미지명:tag\> 로 컨테이너 이미지를 로컬파일로 저장합니다.
my_python.tar 파일이 생성된 것을 확인할 수 있습니다.

## 2. Docker 이미지 불러와서 다른 환경에서 사용하기

### 2-1. Docker 이미지 로드
저는 적용한 서버 환경은 리눅스이나 글 작성 당시 서버를 복구하지 못하여 mac에서 진행합니다.
`이미지를 통한 작업을 해보시고 Dockerfile을 만들어서 Dockerfile을 통한 build로 진행하시는 것을 권유드립니다.`


```bash
docker load -i my_python.tar
```
docker load -i(read from tar archive file) \<tar file\> 명령어로 로컬파일을 docker 이미지 파일로 불러옵니다.

```bash
docker images

REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
my_python    3.8.13    4a39243501c2   26 minutes ago   1.39GB
```
## 2-2. Docker 이미지 컨테이너화와 접속
이미지가 잘 불러와진 것을 확인할 수 있습니다.
이제 이미지를 컨테이너로 실행하고 접속해보겠습니다.
```bash
docker run -d -it my_python -p 10880:8888 -v $(pwd):/home/work
```
> 옵션 설명
-v : volume 의 약자로 \<사용자 디렉토리\>:\<컨테이너 디렉토리\> 를 연동

```bash
docker exec -it my_python bin/bash
```
을 통해 cli 접속 해보면 잘 되는 것을 확인할 수 있습니다.
이제 전과 동일하게 가상환경 접속하여 jupyter lab을 실행해 줍니다.

### 2-3. 쥬피터 서버 실행
```bash
cd home
source .venv/bin/activate
nohup jupyter lab --ip 0.0.0.0 --allow-root &
```
실행 후 포트포워딩 된 ip로 접속합니다.

### 2-4. 외부 접속 확인

![](blog_img/2022_10_31_Docker_jupyter/jupyer_lab_ext_con.png){: .normal}

포트포워딩 된 IP로 mac 쥬피터 서버에 접속한 것을 확인할 수 있습니다.

## 3. 마무리
Docker image 생성 후 불러와서 적용해보는 튜토리얼로써 각자 입맛에 맞게 활용해보시기 바랍니다.
