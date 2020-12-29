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
   - 하고 Build image 하면 됨(처음엔 시간 좀 걸림)
   - 다시 포테이너 Images탭 들어가면 두개의 이미지 생겨있음(만든거랑 python)

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