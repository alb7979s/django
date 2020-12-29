##### 도커를 사용할 겁니다.



##### 도커 사용하는 이유

- 가상화(virtualization)할 때 빠르대(아래 그림 왼쪽이 도커 오른쪽이 일반 가상화)

![img](https://blog.kakaocdn.net/dn/cyQfVA/btqRomr4irk/bRV6njFLsO9S5Nemy6dApk/img.png)

- 어떤 운영체제든 같은 환경으로 사용 가능
- 자기 작업환경 이미지라는 형태로 저장해두고 이 이미지를 원할때마다 실제 서버 구축할때 사용할 수 있음
- 서버 옮기거나 해야할 때 이미지 하나로 커버 가능
- 장점: 거의 모든 곳에서 사용, 빠름

![img](https://blog.kakaocdn.net/dn/cnD6Xp/btqRqQeBPzY/FfNEzKpbs0y7TiDR7UIvM0/img.png)

- 이런 느낌이래 이미지-콘테이너 관계



##### VPS(Virtual Private Server) 대여

- 실제 서버 빌리는건 비싸니까 이 서버의 일정한 자원을 빌리는게 VPS

- [vultr](https://www.vultr.com/?ref=8741024-6G) 들어가서 가입하면 100불 크레딧 준대
- 로그인 했다 가정하에
- +버튼 눌러서 Cloud Compute 누르고 Seoul, 다 나갔으면 다른곳으로
- Server Type은 Application - Docker - Ubuntu18.03 사용할거(우분투에 도커 설치된 상태)
- Server Size는 5달러 짜리(켜놓을때만 과금된대 안사용할땐 삭제해놓음 된대)
- 눌르면 설치중 뜨다가 좀 있으면 서버 켜짐
- Cloud Instance 눌러서 들어가기
- 거기 있는 주소로 접속해서 사용하면 됨

##### 접속)

- cmd - ssh (안되면 openssh 설치 후 하면 됨)
- ssh root@IP
- password 치면 접속 됨



##### docker

- docker 쳐보면 설치 되었나 볼 수 있음

- docker container ls 해보면 돌고있는 도커 확인 가능

- dockerhub:: 전세계 모든 도커 이미지 올리고 받고 할 수 있는곳
- portainer.io 부터 받을거, 도커 GUI로 바꿔주는 SW임
- [dockerhub](https://hub.docker.com/) 들어가서 portainer 검색해서 -ce 붙은거 사용해야함, 위에껀 더이상 지원 안해준대
- 들어가서 deploy portainer 눌러보면 구동하는 방법 나옴

```
docker volume create portainer_data

 docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce

docker container ls 쳐보면 구동되고 있는 컨테이너 (portainer) 나옴
```

- 이제 브라우저에서 빌린 IP:9000 들어가면 portainer 들어가짐
- 관리 비밀번호 설정하고 시작하면 들어가짐 docker 누르고 connect하면 됨
- local - container 눌러보면 portainer 돌아가고 있음을 볼 수 있음

##### Port의 이해 & Nginx 컨테이너 생성

- port: 서버와 통신할 때 사용하는거(컴퓨터와 다른 외부 통신) 

![img](https://blog.kakaocdn.net/dn/vaKNJ/btqRCVOjJYn/EUwU0JWck0cDvX85xY92k1/img.png)

- 이런식으로 port 연결해서 통신함

- 80번 포트가 default
- portainer의 왼쪽 탭 Container - Add Container
- name 아무거나, Image에 nginx 입력
- publish a new newtwork port 누르면 port 연결 가능
- host, container에 80, 80 넣고(테스트 할거라) deployment in progress 누르면 컨테이너 생성됨
- 빌린 가상서버 IP로 테스트 해보면 nginx 환영구 나옴

##### docker에 django 올리기

![img](https://blog.kakaocdn.net/dn/nRLmg/btqRQTnZMqe/BHXnzQfe65Vn88KDR5eTD1/img.png)

1. Github에 업로드 하기

   - 가상환경에서 어떤 라이브러리 설치했었는지 정보 남기기

   - 파이참 가상환경 터미널에서 pip freeze > requirements.txt 하면 라이브러리, 버전 이름 적혀나옴

   - 이것도 깃에 같이 올려두기

2. Dockerfile 작성

   - Dockerfile: 어떻게 이미지를 만들지 내용들을 적어주는거
   - FROM:: base이미지를 뭘 쓸지
   - RUN:: 커맨드를 실행 시켜줌(pip install, git clone,  ... )
   - WORKDIR:: cd랑 비슷, 새로운 폴더로(절대경로)
   - EXPOSE:: 포트를 노출시켜서 다른 포트와 연결할 수 있도록 해주는거
   - CMD:: 컨테이너 실행 할때마다 돌아야되는 커맨드 넣어줄거(Ex. python manage.py runserver, 물론 nginx쓰면 다른 커맨드 넣을거긴함)

   - 요기 아래부터 실습 내용
   - pycharm켠 후, 루트폴더에 Dockerfile 만듦

   ```
   #Dockerfile
   FROM python:3.9.0
   
   WORKDIR /home/
   
   RUN git clone https://www.github.com/alb7979s/pinterest.git
   
   WORKDIR /home/pinterest/
   
   RUN pip install -r requirements.txt
   
   #이 줄은 임시로 하는거
   RUN echo "SECRET_KEY=..." > .env
   
   RUN python manage.py migrate
   
   EXPOSE 8000
   
   CMD ["python", "magage.py", "runserver", "0.0.0.0:8000"]
   ```

   - docker hub 가서 python 치고 들어가서 Tags 탭 누르면 어떤 버젼 (사용 할 수)있는지 볼 수 있음

3. 이미지 생성

   -  포테이너 들어가서 Images - Build a new image - Upload - select file해서 Dockerfile 넣어줌 (이름 정해줌 django_test_image:1)
   -  하고 Build image 하면 됨(처음엔 시간 좀 걸림)
   -  다시 포테이너 Images탭 들어가면 두개의 이미지 생겨있음(만든거랑 python)

4. 컨테이너 실행

   - 이러고 Containers - add container 해서 name: django_container 하고 Image: djago_test_image:1 하고 port 8000, 8000 한 후 Deploy the Container 해보면 잘 돌아감

   - 이러고 빌린 아이피:8000 해보면 만든 홈페이지 화면 잘 나옴

##### Gunicorn 설치 및 runserver 명령어 대체

- 위에처럼 runserver로 장고 컨테이너 그대로 실행하면 문제가 있음
- 하면 안되는 방법, 장고 공식문서에 이 명령어 쓰지 말라고 되어있음(개발 테스트용이래)
- 따라서 추가적으로 프로그램을 설치해줄거(Gunicorn)
- gunicorn: nginx라는 웹서버와 django 컨테이너를 연결시켜주는 인터페이스
- 파이참 켜서 pip install gunicorn
- pip freeze > requirements.txt
- docker.file은 안하고 requirements.txt만 해줄거 따라서 git add requirements.txt
- 해서 커밋해줌 "git commit -m gunicorn install update"

```
#Dockerfile - CMD 부분 수정
CMD ["gunicorn", "pinterest.wsgi", "--bind", "0.0.0.0:8000"]
```

- 근데 만든 커맨드들 캐시가 되어있어서 좀 변화를 줘야함(포테이너 써서 그런거래)
- 따라서 echo 위에 RUN pip install gunicorn 명령어 추가
- django_test_image:2로 select file - Dockerfile로 이미지 만든 후
- django_container_gunicorn, django_test_image:2, port: host-8080, container-8000으로 컨테이너 만들면 됨
- 빌린주소:8080으로 접속해보면 나오긴 하는데 원래 했던거처럼 잘 안나옴
- f12해보면 base.css 이런거(정적인 파일들) 못가져옴. 그래서 nginx라는 서버를 앞단에 연결시켜줘서 이 문제를 해결해줄거임

##### Docker Network의 이해 및 구현

![img](https://blog.kakaocdn.net/dn/cHuR7M/btqRL4p9U1S/8NWY45LZYhgEFU9OTj5uP1/img.png)

- nginx -> django로 넘겨주는 과정을 docker network를 이용해 해결해줄거
- network를 만들어 그 내부에서 container name을 통해 통신함

![img](https://blog.kakaocdn.net/dn/bmCYJw/btqRCVt1byK/Qooh0f2sM6KmcZgP2jK56K/img.png)

- 위처럼.
- 포테이너 - containers 들어가서 portainer 빼고 다 지워줌
- Networks 들어가서 nginx-django로 만들어줌
- Containers 들어가서 django_container_gunicorn, django_test_image:2로 만들고 port 없는 채로 (user -> nginx -> django로 넘겨줄거기 때문에 장고에서 외부포트 연결해 줄 필요 없음)

- 아래 Network 부분에 nginx-django 써주고 Deployment 해줌
- 이번엔 nginx 컨테이너 만들어줄건데, 설정파일부터 만들어줘야함
- 파이참 들어가서 루트에 nginx.conf 파일 만들어줌
- [gunicorn](https://gunicorn.org/#deployment) 여기서 내용 복붙해서 수정할거

```
worker_processes auto;

events {
}

http{
    server {
        listen 80;

        location / {
            proxy_pass http://django_container_gunicorn:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
      }
  }
```

- [파일질라](https://filezilla-project.org/) 가상서버 안에 파일들 올리거나 내려줄거
- 파일질라 깔고 host에 IP주소 넣고, 사용자명 넣고, 비밀번호 넣고 포트 22로 연결하면 됨
- .. 눌러서 home 들어가서 django_course폴더 만들어줌
- 이 안에다가 로컬에 있던 파일 nginx.conf 올려줌
- 이제 nginx 컨테이너 만들어줌(image nginx, host-port 80, network-nginx-django, Volums-container에 /etc/nginx/nginx.conf 넣고, Volums-host부분에 /home/django_course/nginx.conf 하고 Bind 체크하고 Deploy 해줌)
- 참고로 위에 volumes에서 컨테이너 안이랑 밖에 있는 host랑(내가 빌린 가상서버) 연결해주는 과정
- 빌린 IP주소 넣으면 들어가짐

##### static의 이해

- static file이란?: 초기에 서버들 단순한 html 파일들, 즉 정적인 파일들만 제공해주는 서버였음(모두 스태틱). 근데 그러다보면 파일들이 너무 많아짐 따라서 static + dynamic으로 변화하게됨 
- 예전엔 서버에서 (static + dynamic) 다 생성해서 클라이언트에 보냈었음. 근데 이게 또 복잡해지다 보니까 동적인 컨텐츠를 따로 생산해낼 수 있는 부분을 분리함

![img](https://blog.kakaocdn.net/dn/u7SzP/btqROesuH9l/dJ3nyKEGncHnY78AOzmAY1/img.png)

- static파일 요청하면 그냥 서버에서 주고 dynamic파일 요청하면 위처럼 과정을 거침
- 따라서 static이랑 dynamic 서브인 하는 부분이 다름
- 지금 만드는 과정을 대입해보면 아래와 같음

![img](https://blog.kakaocdn.net/dn/blJuhU/btqRQSvXDJa/qU7xtQiWVccwvvcZjIoRg0/img.png)

- 따라서 gunicorn - django 에서 static 파일 왜 제공 못받냐면 gunicorn 이랑 django는 동적인 콘텐츠를 제공하기 위한 거라서
- 그래서 이렇게 하려면 static content를 어떻게 해야하나?
  1. 장고 컨테이너안에 있는 static content 한곳에 수집
  2. 그 static content를 nginx container와 동기화

##### static file 취합

- python manage.py collectstatic 명령어를 쓰면 되는데, DOCKER 명령어로 해줄거임. DOCKER 파일에 RUN python manage.py collectstatic 추가
- 그러고 portainer - Images에서 django_test_image:3으로 만들어줌

##### Docker Volume의 이해

- 다른 컨테이너 같의 있는 데이터를 공유할 수 있는 기능

![img](https://blog.kakaocdn.net/dn/ccv0ZK/btqRQRX6hco/9gxGbtEaQpM1zKZ7xLvx61/img.png)

- Bind Volume: HOST, DOCKER 안에 있는 파일이나 폴더 연동시켜서 같이 사용할 수 있도록

![img](https://blog.kakaocdn.net/dn/b9arQN/btqRQRRkoPr/KFqJgHTKfwmA6rnffMKi10/img.png)

- Named Volume: Docker 안에서 볼륨을 하나 만듦, 이 볼륨을 컨테이너에 붙여서 동기화 할 수 있음, 특징은 연결되어 있던 컨테이너가 모두 사라져도 볼륨은 그대로 있음

##### ![img](https://blog.kakaocdn.net/dn/buJ5OI/btqRAkAUMHB/0ou2B5WI0rCk0RypX3d1lk/img.png)

- 그래서 앞으로 이런식으로 만들거

##### Docker Volume 생성 및 Container 적용

- 포테이너 - containers 다 지워줌(포테이너 빼고)
- volumes 들어가서 add volume누르고 static이란 이름으로 만들어줌
- media 라는 볼륨도 하나 만들어줌
- containers 가서 add container누르고 이름django_container_guricorn 으로, Image는 django_test_image:3, network port안건드려도 되고, 밑에 network에서 nginx-django 선택하고, vloumes에서 map additional volume 두개 추가
- 그 볼륨에서 container: /home/pinterest/staticfiles/ 
- volume: static - local
- container: /home/pinterest/media/
- volume: media - local
- 이러고 deployment 해주면 됨
- 컨테이너에서 Quick actions 제일 왼쪽 누르면 로그 확인할 수 있음
- nginx 컨테이너 만들어줌, 이미지 nginx:latest, host port 80, container port 80, network는 nginx-django
- volumes가서 세개 추가해주고 container: /data/static/ , volume: static - local
- container: /data/media, volume: media - local
- container: (Bind누르고) /etc/nginx/nginx.conf, host: /home/django_course/nginx.conf
- deploy하기 전에 고쳐줘야 할게 있어!

```
#nginx.conf - server{} 내부에 추가

include mime.types;

location /static/ {
	alias /data/static/;
}
location /media/ {
	alias /data/media/;
}
```

- 파일질라 이용해서 수정한 파일 올려줌.
- 그러고나서 deploy 하면 됨
- 빌린 IP로 들어가보면 잘~~~ 나옴 개굳

##### Maria DB 컨테이너를 이용한 DB 분리

- docker hub에서 mariadb 치면 나옴 거기서 시작법대로 해주면 됨
- 포테이너 들어가서 Add container로 mariadb 하고 Image에 mariadb:10.5
- 참고로 Image 이름에 :이거 뒤에는 태그값 넣는거 안적으면 그냥 최신으로 가져옴(권장하진 않는대)
- Env에서 MYSQL_ROOT_PASSWORD, password 넣어주고 deployment해보면 되는걸 볼 수 있다

##### 개발/배포 환경 분리

![img](https://blog.kakaocdn.net/dn/nRJlp/btqRI80SVTh/AOCgLCuEvtkLTXUuTVNou0/img.png)

- 요런식으로(mariaDB아래 볼륨을 통해서 컨테이너 생애주기와 관련없이 데이터 유지되도록 하겠)
- 파이참 켜서 config - settings.py 이 설정들 나눠줄거임
- config에 settings라는 폴더 만들어주고, settings.py 이쪽으로 넘겨줌
- settings 폴더 안에 local.py, deploy.py 추가(local은 컴퓨터 개발환경, deploy는 배포환경)
- settings.py 이름을 base.py로 바꿔줌
- base.py에서 같은건 냅두고 다른걸 빼낼거임
- env관련 ~ ALLOWED_HOSTS부분 까지 잘라내서 local.py랑, deploy.py에 넣어줌
- 둘 다 import os, environ랑 from .base import* 해줌
- base.py에서  DATABASE 부분도 잘라서 두 곳에 넣어줌
- local.py는 그대로 냅두면 되는데 deploy.py 에서는 바꿔줘야함
- DEBUG = False로 바꿔주고
- DB 주석부분 컨트롤 클릭 해보면 양식 나옴

```python
DATABASES ={
	'default': {
        'ENGINE': 'django.db.backends.mysql',	#mariaDB가 mysql에서 나온거라
        'NAME': 'django',
        'USER': 'django',
        'PASSWORD': 'password123',
        'HOST': 'mariadb',				
        'PORT': '3306',							#mysql이 사용하는 포트가 3306임
    }
}
```

- 이러고 python manage.py runserver하면 에러남
- 그 이유인 즉슨, settings.py 파일의 위치가 변했기 때문! 따라서 base.py - BASE_DIR에 .parent 추가해줌
- 하지만 또 에러가 나는데 manage.py - main() 부분 pinterest.settings.local로 바꿔주면 잘 작동함

##### Maria DB 컨테이너 설정 및 Django 연동

- database로 해서 볼륨 하나 만들어 놓음

- 포테이너에서 원래 있던 마리아DB 컨테이너 삭제해줌
- 참고로 config - settings - deploy.py 에 있는 DB내용들이랑 맞춰서 적어줘야함
- 이름 mariadb, Image: mariadb:10.5, Network: nginx-dajngo, Volumes: container: /var/lib/mysql(mariaDB 공식 문서 where to stor data 부분에 있는 경로), vloume: database-local
- Env 4개 추가 (참고로 공식 문서에 Environment Variables라고 환경변수 어떤 이름으로 넣어줘야는지 나와있음) 
- MYSQL_ROOT_PASSWORD, password123
- MYSQL_DATABASE, django
- MYSQL_USER, django
- MYSQL_PASSWORD, password123
- 이러고 deployment 해줌
- 이번엔 django container와 연결
- Dokerfile파일 열고 RUN echo "testing" 추가 (의미없음, 깃 수정했는데 그거 캐시 때문에 반영 안되는거 막으려고)
- RUN ~migrate 지우고 밑에 CMD로 추가해줄거임
- CMD에 넣어야하는 명령어가 두개가 됨. 이런 경우에는 다음처럼 바꿈
- CMD ["bash", "-c", "python manage.py migrate --settings=pinterest.settings.deploy && gunicorn pinterest.wsgi --env DJANGO_SETTINGS_MODULE=pinterest.settings.deploy --bind 0.0.0.0:8000"] 이러면 되는데 manage.py 가면 위치가 로컬환경으로 되어있음 따라서 배포 환경으로 바꿔줘야함 --settings=pinterest.settings.deploy가 그 부분, 뒤에 gunicorn 부분은 [여기](https://docs.gunicorn.org/en/latest/run.html#django) 참고함
- 추가로 RUN pip install gunicorn 아래에 RUN pip install mysqlclient 추가해줘야함
- 포테이너 - 이미지 만들기 누르고 Dockerfile 넣어서 django_test_image:4 만들기
- add container 해서 name:django_container_gunicorn, docker: django_test_image:4, network: nginx-django 해서 deployment 하면 됨
- 접속 해보면 잘 돌아감 굿굿

- 글 올리고 장고 컨테이너 없애고 다시 켜봐도 데이터 그대로 남아있음

##### container의 한계, docker stack의 이해

- 설정 계속 해줘야함, 컨테이너 꺼졌을때 문제

![img](https://blog.kakaocdn.net/dn/bdiwkr/btqRGo36i5p/IbDrTBjPy6kM1uGFKftuNk/img.png)

- 따라서 컨테이너별로 들어가는 모든 셋팅들을 하나의 파일로 만들어줄거(해놓으면 컨테이너 하나하나 일일이 셋팅할 필요 없어짐) 	=> Docker STACK 이라고함
- 컨테이너 꺼졌을 경우 reboot 해줘야는디 누가해줘?

![img](https://blog.kakaocdn.net/dn/b8KPgM/btqRAjPJMyC/7cwDnMNYtBKG6eFxugefhK/img.png)

- 이렇게 서비스로 관리할거
- 서비스는 만약 문제 생겨서 없어지면 자동으로 재부팅 시켜줌
- 추가로 서비스 내에서 필요에 따라 컨테이너를 늘리거나 줄일수 있음(scale out 가능)

##### docker swarm의 이해

- 도커시스템을 포함하고 있는 가상환경을 노드라고 하는데, 이 노드가 여러개 있을 수 있음. 이 여러가지 노드를 하나의 서버처럼 묶어주는게 swarm

![img](https://blog.kakaocdn.net/dn/cfTeof/btqRTR4Oyjl/SXLKKpzzG9moUovprDQoPK/img.png)

- 이런식으로 클러스터링 가능

![img](https://blog.kakaocdn.net/dn/pym23/btqRTSvTS1f/O4Y11c1MrPaC9vxrh3jv6K/img.png)

- 필요한 컴퓨팅 파워에 따라 이렇게도 구성 가능(이런걸 container orchestration이라한대)

##### stack을 위한 yml 파일 작성

- portainer에서 container들 다 지워주고
- docker swarm 모드 켜줘야함, 근데 portainer 지금버전에선 제공 안해줘서
- 터미널로 해줘야함

```
ssh root@IP주소
비밀번호

docker swarm init
```

- 이러고 포테이너 들어가면 swarm, services이런거 생겨있음

- 파이참 키고 루트 - docker-compose.yml 생성

```python
#docker-compose.yml
version: "3.7"
services: 
    django:
        image: django_test_image:3		
        ports:
            -8000:8000
```

- 이런식으로 쌓아나가면 됨
- 포테이너 - Stacks 만들고 name:django_stack, docker-compose.yml 파일 업로드해서 deploy
- 이러고 접속해보면 돌아감(장고만 돌려서 스태틱 파일은 안불러진 상태로)
- portainer - swarm보면 어떻게 돌아가는지 알 수 있음, services에서 숫자 올려주면 컨테이너 늘려줄수도 있음

##### 모든 컨테이너 yml 작성

```python
#docker-compose.yml
version: "3.7"
services: 
    nginx:
        image: nginx:1.19.5
        networks:
            -network
        volumes:
            - /home/django_course/nginx.conf:/etc/nginx/nginx.conf
            - static-volume:/data/static
            - media-volume:/data/media
        ports:
            - 80:80
	django_container_gunicorn:
        image: django_test_image:4
		networks:
            - network
        volumes:
			- static-volume:/home/pinterest/staticfiles
            - media-volume:/home/pinterest/media
	mariadb:
        image: mariadb:10.5
		networks:
            - network
        volumes:
            - maria-database:/var/lib/mysql
		environment:
            MYSQL_ROOT_PASSWORD: password123
            MYSQL_DATABASE: django
            MYSQL_USER: django
            MYSQL_PASSWORD: password123

networks:
    network:
        
volumes:
    static-volume:
	media-colume:
	maria-database:
        
```

- 이 파일을 기반으로 배포를 해보자~
- portainer - stacks 가서 원래 있던거 지우고 container들도 지우고
- stack 추가 누르고 name:DJ, .yml파일 업로드 해서 만들어줌
- services가서 다 동작 되면 IP주소 치고 테스트 해볼수있음
- ~~뭔가 엄청난걸 배우고 있는거 같애~~

- 근데 container에 몇개 안돌아가는데 이거 로그 살펴보면 gunicorn 못찾았다고 나옴, 이게 장고가 돌아가기 위해서는 mariadb가 구동되어있어야함 근데 구동 안되고 있으니까 계속 컨테이너 만들기를 시도 하다가 mariaDB되면 돌아감, 또 nginx는 장고가 돌아갈때까지 시도했다가  돌아감
- 따라서 container에 stopped 되어 있는 컨테이너들은 없애도 됨
- tip] 켜지지 않았을때 생성시도를 하지 못하게 하는 방법도 있긴함(고오오급 방법)

##### Docker Secret을 이용한 보안

- django -secretkey, mariadb - password 이런걸 도커에서 관리하면서 필요한 서비스에 제공하도록 할 수 있음
- 포테이너 - secrets 추가 눌러서 name:DJANGO_SECRET_KEY 해서 Secret 부분에 키 넣어줌
- Dockerfiles에 SECRET_KEY 부분 지워줌
- docker-compose.yml에 있는 MYSQL_ROOT_PASSWORD, MYSQL_PASSWORD도

- compose.yml파일 수정할거

```python
#.yml - django_container_gunicorn 부분에 추가
secrets:
    - MYSQL_PASSWORD
    - DJANGO_SECRET_KEY
#.yml - mariadb 부분에도 추가
secrets:
    - MYSQL_PASSWORD
    - MYSQL_ROOT_PASSWORD
#밑에 environment 부분 수정
MYSQL_PASSWORD_FILE: /run/secrets/MYSQL_PASSWORD
MYSQL_ROOT_PASSWORD_FILE: /run/secrets/MYSQL_ROOT_PASSWORD
#마지막 부분에 추가
secrets:
    DJANGO_SECRET_KEY:
        external: true
	MYSQL_PASSWORD:
        external: true
	MYSQL_ROOT_PASSWORD:
        external: true
```

- .yml에 django_test_image:5로 변경

```python
#deploy.py 윗부분에 선언
def read_secret(secret_name):
    file = open('/run/secrets/' + secret_name)
	secret = file.read()
    secret = secret.rstrip().lstrip()
    file.close()
    return secret
#SECRET_KEY 부분 수정
SECRET_KEY = read_secret('DJANGO_SECRET_KEY')
#DATABASES - PASSWORD부분 수정
'PASSWORD': read_secret('MYSQL_PASSWORD'),
```

- 이러고 deploy.py 깃에 올려줌
- 이제 이미지 만들어야는디 Dockerfile가서
- RUN echo 부분에 1234로 변화주고
- RUN python.manage.py collectstatic 뒤로 밀어줄거
- CMD["python manage.py collectstatic --noinput --settings=pinterest.settings.deploy && python ~"] 처럼 추가
- 포테이너 - 이미지 추가 name: django_test_image:5, Dockerfile올려서 이미지 생성
- stacks에서 추가 name:DJ, .yml파일 업로드 해서 생성
- 기다렸다가 다 켜졌음 containers에 stopped 된거 없애주고
- 테스트 해보면 잘 나옴~ 와 완강~~~

