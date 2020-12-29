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

