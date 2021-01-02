##### 핀터레스트를 만들겁니다.

- 글, 마이페이지, 반응형, 댓글, 팔로우, 구독, ... 등등 만들 예정
- 프론트엔드 - JS, HTML, CSS
- 백엔드 - NGINX(서버), django(프레임워크), docker(배포), 마리아(DB)

![img](https://blog.kakaocdn.net/dn/QERpw/btqQjob2AmK/HcvryVTiY0UMJHatNagaK1/img.png)

요렇게 만들면 이 장고 서비스 하나가 도커 - 하나의 컨테이너가 됨

![img](https://blog.kakaocdn.net/dn/UusRc/btqQhK7v2Tj/iJI2d2Itn3qO2ixoNZNiT1/img.png)

이렇게 집어넣어줌. 이런식으로 각각의 컨테이너를 구축해서 도커 시스템으로 구축

마지막에 VULTR 라는 가상 서버 빌린 후 구축한 도커 시스템 올려서 서비스를 배포 해볼거에요!



##### 개발환경 셋업

==> [점프투장고](https://wikidocs.net/72377) 참조하기, 이건 가상환경 가서 django같은거 설치하고 플젝 생성이고

아래건 실제 환경에 django 설치하고 추가한거(이건 venv(가상환경)가 플젝 파일에 자동으로 생성)

- mkdir NAME
- cd NAME
- django-admin startproject config . 
- 그 플젝 열고 인터프리터 가서 add 하고 ok 눌러주면 새로운 가상환경 추가됨

##### MVC(Model View Controller)

- 애플에서 만든 개발 패턴인데 장고는 여기서 controller대신 template으로 (거의 명칭만 다름)

- Model: DB관련
- View: 유저-서버 통신(인증, 계산, 확인 등)
- Template: 인터페이스(FE와 관련있음 HTML같은 정적인 파일을 동적으로 작동하게끔 해줌)
- template <-> view <-> model

##### Acoount

- 터미널 들어가서 (ctrl + shift + tap + t)
- python manage.py startapp accountApp
- project/settings.py 에서 installed_apps 에서 'accountApp' 매핑 시켜줌

```python
#accountApp/views.py
def hello_world(request):
    return HttpResponse('Hello world')	#Alt + Enter 누르면 자동으로 import해줌 개꿀
```

- config - urls.py(가장 먼저 요청 받는 곳) 여기서 url을 연결해줌, 주소창에 쓰는 내용이랑 어디 파일을 참고할건지 연결해주는 작업

```python
#config/urls.py
path('account/', include('accountApp.urls'))	#이거 추가해서 매핑시켜줌
```

- 여기에서 똑같이 연결해주는 작업 해줌

```python
#accountApp/urls.py
app_name = 'accountApp'	#이렇게 해두면 나중에 "127.0.0.1:8000/account/hello_world/" 이렇게 써야될걸 함수써서 "accountApp"으로 호출가능
urlpatterns = [
    path('hello_world/', hello_world, name='hello_world')	#접근주소, view, name
]
```

- 이러고 서버 구동 시킨 후 http://127.0.0.1:8000/account/hello_world/ 들어보면 잘 뜸

##### git

- version control이 핵심

![img](https://blog.kakaocdn.net/dn/cJFQhW/btqSdtbpkLm/YkhAlePiAxE6rj2mSpv8dK/img.png)

- branch로 버전 관리 가능(배포 버전, 베타 버전 이런식으로)
- 위처럼 Merge도 가능

##### git ignore(7강 참고)

- project - .gitignore 파일 만들고 [요기](https://github.com/github/gitignore/blob/master/Global/JetBrains.gitignore)에서 복붙 해넣으면 필요없는거 추적 안함(가상환경  넣을필요 없으니까 이 파일에 venv/도 추가)
- config/settings.py 에 secret_key 관리해줘야하는데, [요기](https://django-environ.readthedocs.io/en/latest/)서 secret_key 노출 안시키고 내부 파일에서 자동으로 읽어서 개선해줄 수 있음
- 위 사이트 installation에서 하란대로 하면 됨(아래 적어놓은대로 하면 됨) 
- pip install dajngo-environ

```python
#settings
import environ
env = environ.Env(
    # set casting, default value
    DEBUG=(bool, False)
)
```

- root폴더에 .env 파일 만들어서

```python
#.env
DEBUG=on
SECRET_KEY=your-secret-key			#이부분 settings에 있는 내 secret_key 입력(''x)
DATABASE_URL=psql://urser:un-githubbedpassword@127.0.0.1:8458/database
SQLITE_URL=sqlite:///my-local-sqlite.db
CACHE_URL=memcache://127.0.0.1:11211,127.0.0.1:11212,127.0.0.1:11213
REDIS_URL=rediscache://127.0.0.1:6379/1?client_class=django_redis.client.DefaultClient&password=ungithubbed-secret
```

```python
#settings
import os
#BASE_DIR아래쪽에
environ.Env.read_env(
    env_file = os.path.join(BASE_DIR, '.env')
)
```

```python
#settings 가서 이렇게 수정하고
SECRET_KEY = env('SECRET_KEY')
#.gitignore 가서
.env 	#라고 추가해주면 추적 안됨
```

- VCS - Enable Version control integration 누르고 git 활성화 됨
- git status 해가면서 필요 없는거 추가해주면 됨 (Ex. .idea/)

```python
#.gitignore
.env
db.sqlite3
.idea/
__pycache__/
#가상환경 python에서 그냥 만들었으면 venv/ 추가
```

##### template(extends, include 둘 다 HTML 가져오는 느낌은 비슷한데 용도가 살짝 다름)

- extends: 바탕을 깔아주는 느낌(구역을 나눠놓은 HTML 파일 가져옴)

![img](https://blog.kakaocdn.net/dn/UrTQz/btqQu4dP7X7/AaEjIiB1hKj30qnZNmQq10/img.png)

- include: 조각들을 가져와서 붙이는 느낌

![img](https://blog.kakaocdn.net/dn/xe51n/btqQmQujZen/SDdFPPj7v0c3ExWZIsZb9K/img.png)

![img](https://blog.kakaocdn.net/dn/bKaDgC/btqQu5cHWuy/XYg7Z1reb2rjmkzFwBCEIk/img.png)

- project 폴더에 templates 폴더 생성, 그 안에 base.html 생성
- views.py에서 html파일(템플릿) 가지고 와서 그 내용 채워넣는 형식으로 만들거임
- 기본 동작: base.html이 temlates 안에 있음으로써 accountApp-view에서 응답을 해줄 때 여기서 템플릿을 가져와서 내용을 나타내는 역할, 그니까 view에서 템플릿을 가져와서 거기에 내용을 채워넣는 식으로 만들거임

```python
#views.py 
def hello_world(request):
    return render(request, 'base.html')		#요게 가져오는거
```

- 근데 이러고 테스트 해보면 에러뜸, config - settings.py에서 templates 경로 입력 안해줘서 그럼

```python
#config/settings.py 에 매핑 해줘야함
TEMPLATES = [
    {
        'DIR': [os.path.join(BASE_DIR, 'templates')],
    }
]
```

- 이러고 테스트 해보면 잘 됨
- base.html에서 우선 head 부분 새로 .html 만들고 include 구문으로 가져오도록 할거임
- templates파일에 head.html 생성

```html
<!-- head.html -->
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
```

```html
<!-- base.html -->
<!DOCTYPE html>
<html lang="ko">
    
{% include 'head.html' %}
<body>
    <!-- border-radius: 블럭의 가장자리, margin: 컴포넌트 사이 마진?-->
    <div style='height: 10rem; background-color: #38df81; border-radius: 1rem; margin: 2rem'> 
    </div>
    
    <div style='height: 20rem; background-color: #38df81; border-radius: 1rem; margin: 2rem'> 
    </div>
    
    <div style='height: 10rem; background-color: #38df81; border-radius: 1rem; margin: 2rem'> 
    </div>
    </body>
</html>
```

- ##### Tip] 알트 누르고 커서 여러개 클릭하면 다중커서 조절 가능!!

- div 태그는 가능한 폭을 모두 가져가는 형태

- 서버 켜서 어떻게 되었나 한번 확인해보기

- 여기 body에서 3개 부분이 있는데 첫부분, 세번째 부분은 계속 사용할거므로 include 구문 사용해서 만들어줌

```html
<!-- header.html, footer.html 만들고 아래처럼 include하면 똑같이 쓸수있음-->
<body>
	{% include 'header.html' %}
    
    <!-- content라는 블럭을 새로 만들어서 내용 바뀌는거 처리해줄거 -->
    {% block content %}
    {% endblock %}
    
    {% include 'footer.html' %}
</body>
```

- content부분 accountApp 내부에서 사용할거임, 이제 그거 사용하기 위해
- accountApp 폴더에 templates 폴더 만들고, 그 안에 accountApp 폴더 만들어줌
- 그 안에다가 .html 파일 만들어줄거, 저거 폴더 많이 만드는 이유는 나중에 가져오는 코드 짤때 가독성 높이기 위해서래(사용시 어떤 앱에서 html을 가져왔는지 알 수 있음)
- hello_wolrd.html 만들고 

```html
<!--아래처럼 하면 base.html 내용 다 가져오는데 block만 바꿀 수 있음-->
{% extends 'base.html' %}	

<!-- 아래처럼 내용 쓰면 렌더링 하게 됨-->
{% block content %}
	<div style='height: 20rem; background-color: #38df81; border-radius: 1rem; margin: 2rem'>
        <h1>
            test1
        </h1>
	</div>
{% endblock %}
```

- 그니까 어떤 앱에서 html 페이지를 만들껀데, extends로 뼈대를 가져오고 block으로 된것들 위처럼 하면 입맛에 따라 꾸밀 수 있음

```python
#accountApp/views.py
def hello_world(request):
    return render(request, 'accountApp/hello_world.html')
```

- accountApp/hello_world.html을 가져오도록(그니까 hello_world.html을 가져오는데, 그 html은 base.html(뼈대)를 가져오고, 그 안에 뼈대는 나눠진 부분마다 html을 만들어서 include로 가져오는 형태임)

##### style

- 윗 부분을 꾸며봅시다~

```html
<!--header.html-->
<div>
    <div>
        <h1>pinterest</h1>
    </div>
    <div>
        <span>nav1</span>
        <span>nav2</span>
        <span>nav3</span>
        <span>nav4</span>
    </div>
</div>
```

- nav -> 나중에 네비게이션바 만들거(어느 곳으로 갈지 나타내는거)
- 이거 하고 서버 키고 들어가보면 왼쪽에 가있는데 F12 눌러서 테스트 해볼 수 있음
- div쪽 누르고 style 부분에 text-align: cneter; 이런식으로 테스트 가능
- 이번엔 밑 부분을 꾸며봅시다~

```html
<!--footer.html-->
<div style="text-align:center; margin-top: 2rem;">
    <div style="font-size: .6rem;">
        <span>공지사항</span> |
        <span>제휴문의</span> |
        <span>서비스 소개</span>
    </div>
    <div style="margin-top: 1rem;">
        <h6 style="margin-top: 1rem;">pinterest</h6>
    </div>
</div>
```

- margin: 2rem 0; 이렇게 두 부분으로 나누면 앞에는 상하, 뒤에는 좌우를 나타냄
- base.html \<body\>에 위, 중간, 아래 사이에 \<hr\> 구분선 넣어주면 더 이뻐짐

##### bootstrap

- 트위터에서 만든 스타일 라이브러리 같은거임, css를 링크 거는것만으로 바로 적용시킬 수 있음

- [여기](https://getbootstrap.com/docs/5.0/getting-started/introduction/)들어가서 css 복사 한 후에 head.html \<head\>쪽에 넣어줌
- Tip) ctrl + / 하면 주석
- 추가로 폰트 적용할거
- 구글 폰트 검색해서 글자 써보고 이뻐보이는 폰트 고르면 됨 select this style 누르면 링크 뜨는데 head.html 파일에 복붙해주면 됨(이런거 적용하면서 어떤건지 주석 달기)
- css rules to specify families 부분 복사해서 적용할 곳에 붙여넣기 해넣어주면 끝(적용할 style 부분, 여기선 header.html의 제목부분, footer.html의 아래 제목 부분)

##### CSS파일 분리

- 보통 html에 뼈대만 남겨두고 style 같은거(디자인적인거) css로 빼줌
- static(css, js, font 등 자주 변경되지 않는 파일들) 설정 해볼거

```python
#setting.py
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')	#나중에 python manage.py collectstatic 하면 static 파일 모아주는데 그 모아주는 위치임
STATICFILES_DIRS = [
    BASE_DIR / "static",
]
```

- [파일, 디렉토리 경로 다루는 os 주요 함수 정리](http://pythonstudy.xyz/python/article/507-%ED%8C%8C%EC%9D%BC%EA%B3%BC-%EB%94%94%EB%A0%89%ED%86%A0%EB%A6%AC)

- os.path.join(): 경로를 병합하여 새 경로 생성
- 프로젝트 폴더에 static 폴더 만들어줌 여기서 static 파일들 관리할거
- static에서 base.css 생성

```html
<!-- footer.html -->
<!-- 수정 -->
<h6 class="pinterest_footer_logo">pinterest</h6>
```

```css
/* base.css */
.pinterest_footer_logo{
    font-family: 'Lobster', cursive;
}
```

```html
<!-- head.html -->
{% load static %}	<!-- 맨 위에 -->

<!-- DEFAULT CSS LINK -->
<link rel="stylesheet" type="text/css" href="{% static 'base.css'%}">
```

- css링크가 어떻게 되냐면 settings.py - STATIC_URL 부분이랑 장고가 알아서 합쳐줌(STATIC_URL 바꿔가며 테스트해보기)

```css
/* base.css 이번에 바꿀곳들인데, html은 알아서 찾아 바꾸기 */
.pragmatic_logo {
    font-family: 'Lobster', cursive;
}

.pragmatic_footer_button {
    font-size: .6rem;
}

.pragmatic_footer {
    text-align:center;
    margin-top: 2rem;
}

.pragmatic_header {
    text-align:center;
    margin: 2rem 0;
}
```

##### CSS 핵심

- display

  - block
    - 부모의 최대한의 너비 가지고 블록 모양으로 있는 애
  - Inline
    - 글씨의 높이만큼
  - Inline-block
    -  블록인데 inline처럼 안쪽부터 쌓이는 블록
  - None
    - 없어요

- visibility

  - hidden
    - display가 만약 None으로 바뀌면 빈곳  채워 넣는데 visibility-hidden이면 보이진 않지만 존재함 다른거 그대로

- size

  :Respnsive 즉, 반응형으로 만들어야 하므로 size 중요

  - px
    - 부모에 상관없이 픽셀 크기만큼
  - em
    - 부모가 커지면 그만큼 자기도 커짐
    - 근데 부모가 여러개 있을 때 문제 (Ex.할아버지가 2배, 부모 2배 되면 자신은 4배 됨)
  - <u>rem</u>
    - 부모에 상관없이 root HTML의 사이즈에 따라 바뀜
- 1rem = 16px이 기본
  
  - %
    - 바로 위의 부모만 따라감 

##### display 실습

- style 적용 방법 3가지

```html
<!-- 적용 우선순위 1, 3, 2-->
<!-- 1 -->
<div style=""> </div>
<!-- 2 -->
<div class ="NAME"> </div>
.NAME {
	STYLE
}
<!-- 3 --> 
<style>
    .tesing{
        background-color: white;
    }
</style>
```

```html
<!-- display 실습 -->
<style>
    .testing{
        background-color: white;
        height: 3rem;
        width: 3rem;
        margin: 1rem;
    }
</style>
<div class="testing" style="display: block;"> block </div>
<div class="testing" style="display: block;"> block </div>
<div class="testing" style="display: inline;"> inline </div>
<div class="testing" style="display: inline;"> inline </div>
<div class="testing" style="display: None;"> None </div>
<div class="testing" style="display: inline-block;"> inline-block </div>
<div class="testing" style="display: inline-block;"> inline-block </div>
<div class="testing"> div_default </div>
<span class="testing"> span_default</span>
```

- div 기본 속성 block
- span 기본 속성 inline 
- 이렇게 놀다가 commit 안해도 되니까 git reset --hard HEAD 하면 가장 최근 커밋으로 돌아감

##### Model

```python
#accountApp/models.py
class HelloWorld(models.Model):
    text = models.CharField(max_length=255, null=False)	#null은 공백이여도 되는지
```

- python manage.py makemigrations  (models.py에 쓰는 내용을 DB와 연동시킬 파이썬 파일로 만들어주는 작업)
- python manage.py migrate 해주면 적용됨
- tip) migrations 파일들 지우는거 지양하기 고치려면 복잡해짐

##### HTTP 프로토콜 (GET, POST)

- GET
  - 조회
  - 추가적인 parameter 주소에 같이 보내줌
- POST
  - 무언갈 만들거나 수정할 때
  - 데이터 BODY에 넣어 보냄

##### HTTP 실습

- post 쓸라면 .html 파일 안에 \<form\>을 만들어줌(보낼 데이터 꾸러미라 생각하면 됨)

```html
<!-- hello_world.html -->
<form actions="/account/hello_world/" method="post">	
	<input type="submit" class="btn btn-primary" value="POST">
</form>
```

- 첫 줄은 action은 요청 보낼 주소로 method - "post"를 사용해서 보내겠다 이런뜻
- value => 버튼 안에 들어갈  텍스트
- 이대로 서버 돌려서 버튼 눌러보면 에러뜸

```html
<!-- 장고 쓸때 form 안에 항상 명시해줘야함 -->
{% csrf_token %}
```

- csrf_token은 장고에서 제공하는 보안 기능 중 하나임

```python
#accountApp/views.py
def hello_world(request):
    #요청을 받은 객체 안에서 메소드가 POST일 경우
    if request.method == "POST":
        return render(request, 'accountApp/hello_world.html', context={'text': 'POST METHOD!'})
	else:
        return render(request, 'accountApp/hello_wolrd.html', context={'text': 'GET METHOD!'})
```

```html
<!-- hello_world.html -->
{{ text }}
```

##### POST 통신을 통한 DB데이터 저장

- 중간 초록색 없애주기(hello_world.html 가서 높이, 배경색 지우기)
- 내용 중간으로 모아주기(text-align: center)

```html
<!-- hello_world.html -->
<h1 style = "font-family: 'lobster', 'cursive';">
	Hello World LIST!                                                 
</h1>
<form action="/account/hello_world/" method="post">
    {% csrf_token %}
    <div>
        <input type="text" name="hello_world_input">
    </div>
    <div>
        <input type="submit" class="btn btn-primary" value="POST">
    </div>
    <h1>
        {{ text }}
    </h1>
</form>
```

- font-family: 'lobster', 'cursive'; 뜻이 만약 lobster 없으면 cursive 가져와라 이런 뜻

```python
#views.py
#hello_world() if문
temp = request.POST.get('hello_world_input')
return render(request, 'accountApp/hello_world.html', context={'text': temp})
```

- request.POST중 hello_world_input이라고 받은 데이터를 temp에 저장
- 여기까지 하면 통신해서 데이터 긁어오는거함
- 아래서는 그 데이터 저장하는 거까지

```python
#views.py
#hello_world() if문
temp = request.POST.get('hello_world_input')
new_hello_world = HelloWorld()			#models.py에 만들어놓은 모델 가져옴
new_hello_world.text = temp				#가져온 모델의 text필드에 저장
new_hello_world.save()					#DB에 저장
return render(request, 'accountApp/hello_world.html', context={'hello_world_output': new_hello_world })	#객체를 보내는거임 따라서 아래에 추가 작업
```

```html
<!-- hello_world.html -->
{% if hello_world_output %}
<h1>
    {{ hello_world_output.text }}
</h1>
{% endif %}
```

##### DB정보 접근 및 for loop

```python
#views.py
#new_hello_world.save() 아래쪽에 추가
#(GET, POST 두 부분 모두 아래 두줄 추가)
hello_world_list = HelloWorld.objects.all()	#HelloWorld의 모든 데이터 긁어옴
return render(request, 'accountApp/hello_world.html', context={'hello_world_list': hello_world_list })
```

- hello_world.html 가서 hello_world_output => hello_world_list로

- 이러고 테스트 해보면 오브젝트 통째로 나옴, 보기 좋게 바꿔주려면 아래처럼 수정

```html
<!-- hello_world.html -->
{% if hello_world_list %}
	{% for hello_world in hello_world_list %}
	<h4>
        {{ hello_world.text }}
	</h4>
	{% endfor %}
{% endif %}
```

- 이러고 테스트 해보면 잘나옴, 근데 F5 누르면 계속 반복해서 쌓임 이거 고쳐주려면

```python
#views.py
#post 완료한 이후 get으로 되돌아가서 이런 상황 발생하지 않기 위해서 해주는 작업
#POST부분 return 아래처럼 바꿔줌
return HttpResponseRedirect('account/hello_world/')
#위처럼 해도 되는데 urls.py에서 이름 저장해놓은거 이용해서 바꿔주면
return HttpResponseRedirect(reverse('accountApp:hello_world'))
#바로 윗줄 이름 부분 :다음 공백 넣었다가 오류 났었음
```

##### 디버깅

- 중단점 설정하고

- Run-EditConfigurations-Templates-python
- Script path => 가상환경 폴더 - Scripts로 정해줌
- Parameters => runserver
- 확인하고 나와서 왼쪽에 manage.py 우클릭,  Debug 'manage' 하면 서버 실행되면서 디버깅 가능
- 서버에서 어떤 이벤트 발생시키면서 중단점 때의 상태 볼 수 있음

##### Class Based View, 장고의 CRUD (이론 위주)

- **C**reate **R**ead **U**pdate **D**elete	=> 장고에선 이 네가지 작업 쉽게 할 수 있는 클래스 제공(CBV)
- CBV <-> FunctionBasedView: 지금까지 만든 함수 기반 뷰
- CBV으로 짜면 생산성, 가독성 좋아짐

![img](https://blog.kakaocdn.net/dn/Kxrs5/btqQPzkuojQ/CEf0O0BhXsh7ao3tfI7EnK/img.png)

- 앞으로 이런 형태의 account앱을 만들거임

##### CreateView를 통한 회원가입 구현

```python
#views.py에 추가
class AccountCreateView(CreateView):
    model = User		#tip) ctrl + b 누르면 그 소스코드로 갈 수 있음
    form_class = UserCreationForm
    success_url = reverse_lazy('accountApp:hello_world')
    template_name = 'accountApp/create.html'
```

- 함수랑 class 불러오는 방식때문에 class에서 reverse()쓰면 에러뜸 그래서 reverse_lazy 써준거

```python
#urls.py - urlpatterns에 추가
path('create/', AccountCreateView.as_view(), name = 'create'),
```

```html
<!-- accountApp-templates-accountApp에 create.html 생성 후 -->
{% extends 'base.html' %}
{% block content %}
	<div style = "text-align: center">
        <form action="{% url 'accountApp:create' %}" method = "post">
            {% csrf_token %}
            {{ form }}
            <input type="submit" class="btn btn-primary">
        </form>
	</div>
{% endblock %}
```

- 서버 키고 테스트 해보기 (127.0.0.1:8000/account/create)

##### Login/ Logout 구현

```python
#url.py - urlpatterns에 추가, login, logout은 간단해서 여기에 직접 선언할거임
path('login/', LoginView.as_view(template_name='accountApp/login.html'), name='login')
path('logout/', LogoutView.as.view(), name='logout' )
```

```html
<!-- accountApp-templates-accountApp에 login.html 생성 -->
{% extends 'base.html' %}
{% block content %}
	<div style="text-align: center">
        <div>
            <h4>
                Login
            </h4>
        </div>
        <div>
            <form action="" method="post">
                {% csrf_token %}
                {{ form }}
                <input type="submit" class="btn btn-primary"
            </form>
        </div>
	</div>
{% endblock %}
```

- 주소/accountApp/login 테스트 해보면 잘 나옴

![img](https://blog.kakaocdn.net/dn/bOIqmA/btqQ0O1Zmba/LmC7e6h8C9tTSTXWx7cDSK/img.png)

- Login View가 이런 과정을 거치는데, 위에까지는 next, login_redirect_url 둘 다 안설정해줘서 아이디랑 비번 쳐보면 Default 값으로 가서 에러났었음, 원치 않는 경로로 가지 않도록 아래에 설정해줄거

```html
<!-- header.html - nav 부분 수정 -->
{% if not user.is_authenticated %}		<!-- login 안되어 잇으면 login 보여줌-->
<a href="{% url 'accountApp:login' %}?next={{ request.path}}">
	<span>login</span>
</a>
{% else %}
<a href="{% url 'accountApp:logout' %}?next={{ request.path}}">
    <span>logout</span>
</a>
{% endif %}
```

- login 이나 logout 완성 하고 난 후 다시 갈 경로 next로 받아줌. get방식으로 받은거

- 이렇게해도 accountApp/login 직접 들어가서 치면 default로 넘어가는데, 이거 방지하려고 이번에는 login_redirect_url 설정해줄거

```python
#pinterest - settings.py에 추가
LOGIN_REDIRECT_URL = reverse_lazy('accountApp:hello_world')
LOGOUT_REDIRECT_URL = reverse_lazy('accountApp:login')
```

##### Bootstrap을 이용한 Form 디자인 정리

- pip install django-bootstrap4

```python
#settings.py - INSTALLED_APPS에 추가
'bootstrap4',
```

```html
<!-- login.html 윗부분에 추가-->
{% load bootstrap4 %}
<!-- {{ form }} 부분 수정-->
{% bootstrap_form form %}
<!-- style 부분에 max-width: 500px; margin: 4rem auto 추가 -->
<!-- btn도 수정해줌 "btn btn-dark rounded-pill col-3 mt-3" -->
```

- mt는 margin-top 이고, mt-3은 기존의 3배
- col-6 같은 경우는 12가 100%고 6이면 50%, 3이면 25%(부모 너비의)

```html
<!-- create.html 위 login.html처럼 수정-->
{% extends 'base.html' %}
{% load bootstrap4 %}

{% block content %}

	<div style="text-align: center; max-width: 500px; margin: 4rem auto">
        <div class="mb-4">
            <h4>
                SignUp
            </h4>
        </div>
        <form action="{% url 'accountApp:create' %}" method="post">
            {% csrf_token %}
            {% bootstrap_form form %}
            <input type="submit" class="btn btn-dark rounded-pill col-6 mt-3">
        </form>
	</div>

{% endblock %}
```

- 이번엔 hello_world 부분 폰트 바꿔줄거
- 글꼴 otf 설치한 후에 압축 풀고 .otf 이름 간단한 것들만 복사해서
- pinterest-static에 fonts 폴더 생성해서 복사한 파일들 붙여넣기

```html
<!-- head.html - </head> 윗부분에 아래와 같은 형식으로 4개 추가 -->
<style>
    @font-face {
        font-family: 'NAME';
        src: local('NAME'),
            url("{% static 'fonts/NAME.otf' %}" format("opentype"))
    }
</style>
```

```html
<!-- base.html <body> 부분 수정 -->
<body style="font-family: 'NAME'">
```

- 이러고 account/hello_world 테스트 해보면 잘 됨

##### DetailView를 이용한 개인 페이지 구현(ReadView부분)

```python
#views 부분 추가
class AccountDetailView(DetailView):
    model = User
    context_object_name = 'target_user'
    template_name = 'accountApp/detail.html'
```

```html
<!-- detail.html 만들어서 -->
{% extends 'base.html' %}
{% block content %}
	<div>
        <div style="text-align: center; max-width: 500px; margin: 4rem auto;">
            <p>
                {{ user.date_joined }}
            </p>
            <h2>
                {{ user.username}}
            </h2>
        </div>
	</div>
{% endblock %}
```

```python
#urls.py - urlpatterns에 추가
path('detail/<int:pk>', AccountDetailView.as_view(), name='detail'),
```

- detail에서 특정 유저의 정보를 봐야하니까 그 계정의 PK가 필요, 그래서 <int: pk> 선언해줌

```html
<!-- header.html - else 아래 추가-->
<a href="{% url 'accountApp: detail' pk=user.pk %}">
	<span>MyPage</span>
</a>
```

- 테스트 해보면 logout 상태에서 Mypage안보이고 login 상태면 Mypage 보임
- 접속한 유저의 페이지만 볼 수 있으니까 좀 수정해주면

```python
#views.py - AccountDetailView() 부분에 추가
context_object_name = 'target_user'
```

- detail.html의 user 부분 target_user로 수정

- 이렇게해서 다른사람이 내 페이지 왔을때 내 페이지 보여줄 수 있음

##### UpdateView를 이용한 비밀번호 변경 구현

```python
#views.py에 추가
class AccountUdateView(UpdateView):
    model = User
    form_class = UserCreationForm
    context_object_name = 'target_user'
    success_url = reverse_lazy('accountApp:hello_world')
    template_name = 'accountApp/update.html'
```

```python
#url.py - urlpatterns에 추가
path('update/<int:pk', AccountUpdateView.as_view(), name='update')
```

```html
<!-- update.html 생성 후 -->
{% extends 'base.html' %}
{% load bootstrap4 %}

{% block content %}

	<div style="text-align: center; max-width: 500px; margin: 4rem auto">
        <div class="mb-4">
            <h4>
                Change Info
            </h4>
        </div>
        <form action="{% url 'accountApp:update' target_user.pk%}" method="post">
            {% csrf_token %}
            {% bootstrap_form form %}
            <input type="submit" class="btn btn-dark rounded-pill col-6 mt-3">
        </form>
	</div>

{% endblock %}
```

```html
<!-- detail html 수정 -->
{% extends 'base.html' %}
{% block content %}
	<div>
        <div style="text-align: center; max-width: 500px; margin: 4rem auto;">
            <p>
                {{ target_user.date_joined }}
            </p>
            <h2>
                {{ target_user.username}}
            </h2>
            
            {% if target_user == user %}
            <a href="{% url 'accountApp:update' pk=target_user.pk %}">
                <p>
                    Change Info
                </p>
            </a>
            {% endif %}
        </div>
	</div>
{% endblock %}
```

- 이러고 account/detail/1 테스트 해보면 ChangeInfo 나옴, 그거 누르면 바꿀수있음
- 문제: 아이디 바꿔서 비번 바꾸면 아이디도 바뀜. 따라서 Username칸 비활성화 시켜줘야함

```python
#accountApp(부모) - forms.py 생성
from django.contrib.auth.forms import UserCreationForm
class AccountUpdateForm(UserCreationForm):	#상속 받아줌
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        
        self.fields['username'].disabled = True
```

- 상속 받아서 UserCreationForm = AccountUpdateForm 인데, 마지막줄 추가시켜줘서 초기화 이후에 저 값을 바꿔줌(username 칸을 비활성화 시켜줌) 이런식으로 커스터마이징함

```python
#views.py - AccountUpdateView() 부분 수정
form_class = AccountUpdateForm
```

##### DeleteView 기반 회원 탈퇴 구현

```python
#views.py에 추가
class AccountdeleteView(DeleteView):
    model = User
    context_object_name = 'target_user'
    success_url = reverse_lazy('accountApp:login')
    template_name = 'accountApp/delete.html'
```

```python
#url.py - urlpatterns에 추가
path('delete/<int:pk>', AccountDeleteView.as_view(), name='delete')
```

```html
<!-- delete.html 생성 후 -->
{% extends 'base.html' %}

{% block content %}

<div style="text-align: center; max-width: 500px; margin: 4rem auto">
    <div class="mb-4">
        <h4>
            Quit
        </h4>
    </div>
    <form action="{% url 'accountApp:delete' target_user.pk %}" method="post">
        {% csrf_token %}
        <input type="submit" class="btn btn-danger rounded-pill col-6 mt-3">
    </form>
</div>

{% endblock %}
```

```html
<!-- detail.html 에서 Change Info아래부분에 나타나게 추가-->
<a href="{% url 'accountApp:delete' pk=target_user.pk %}">
	<p>
    	Quit
    </p>
</a>
```

- tip] 저 위에 btn 이런건 bootstrap 홈페이지에 검색해보면 다 나옴

- 이러고 테스트 해보면 아이디 삭제시킬 수 있음

```html
<!-- header.html 부분 SingUp 안만들어서 login밑에 보강해주기 -->
<a href="{% url 'accountApp:create' %}">
	<span>SignUp</span>
</a>
```

##### Bug FIx

- .html - pk=user.pk 이거 pk=target_user.pk로 바꿔줌
- .views.py에 context_object_name = 'target_user' 추가

```python
#config - urls.py - urlpatterns 부분 수정
paht('accounts/', include('accountApp.urls')),
```

##### Authentication 인증시스템 구축 

```python
#views.py - hello_world() 부분에 인증과정 넣을거임
if request.user.is_authenticated:
#    if request.method == "POST": ~
else:
    HttpResponseRedirect(reverse('accountApp:login'))
```

- accounts/hello_world 테스트 해보면 로그인 되어있지 않으면 로그인창으로 가고,
- 로그인하고 위 주소로 가면 hello_world 잘 뜸

- 근데 주소에 accounts/update/1 이런식으로 치면 다른 계정 들어갈 수 있음 이거 로그인 해서 들어갔을때만 가능하도록 바꿔줄거임

- 그냥 로그인해서 들어가도록만 고쳐주면 내 아이디로 로그인하고 저런식으로 다른사람거 들어갈수도 있음 이것도 고쳐줘야함( self.get_object() == self.request.user 부분)

```python
#views.py - AccountUpdateView(), AccountDeleteView() 아래 부분에 추가
def get(self, *args, **kwargs):
    if self.request.user.is_authenticated and self.get_object() == self.request.user:
        return super().get(*args, **kwargs)
    else:
        return HttpResponseForbidden()
    
def post(self, *args, **kwargs):
    if self.request.user.is_authenticated and self.get_object() == self.request.user:
        return super().get(*args, **kwargs)
    else:
        return HttpResponseForbidden()
```

##### Decorator를 이용한 코드 간소화(바로 위 코드들 지워줌)

![img](https://blog.kakaocdn.net/dn/Ua6fG/btqRavOILyI/kk9nTLK4UsBRAN63GGelZK/img.png)

- 이런 Decorator를 사용해서 더 깔끔하게 고쳐줄거임

```python
#views.py - import구문 아래에 추가
@login_required		#이거 추가하고 바로 전 hello_world() 수정했던 if-else문 지워줌
#AccountUpdateView()는 class이고, 안에 함수 메서드 이므로 좀 다르게 선언함
#AccountUpdateView(), AccountDeleteview위에 선언
@method_decorator(login_required, 'get')
@method_decorator(login_required, 'post')
@method_decorator(account_ownership_required, 'get')
@method_decorator(account_ownership_required, 'post')
```

```python
#accountApp(부모) - decorators.py 생성
def account_ownership_required(func):
    def decorated(request, *args, **kwargs):
        user = User.objects.get(pk=kwargs['pk'])
        if not user == request.user:
            return HttpResopnseForbidden()
        return func(request, *args, **kwargs)
    return decorated
```

- 이렇게 할 수 잇는데 이번엔 method_decorator가 너무 많으니까 이거도 좀 수정해주자

```python
#views.py - import 아래쪽에
has_ownership = [account_ownership_required, login_required]
#method 부분 수정
@method_decorator(has_ownership, 'get')
@method_decorator(has_ownership, 'post')
```

- 이런식으로 배열 만들어서 줄여줄 수 있음

##### superuser, media 관련 설정(셋팅)

- python manage.py createsuperuser해서 username, password 설정
- 그 후 서버 켜서 /admin가서 로그인 해보면 접속 됨. 여기서 나머지 계정 싹 지워놓으래

```python
#config - settings.py에 추가
MEDIA_URL = '/media/'		#이 URL 개인적으로 커스터마이징 가능
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

- MEDIA_URL: 주소창에 media/ 이하의 경로로 접근을 해야 실제 미디어 파일에 접근 가능하단 뜻
- MEDIA_ROOT: 미디어파일 서버에 올렸을 때, 어느 경로에 저장 될 것인지
- 이미지 관리할 라이브러리 설치 pip install pillow

##### Profileapp 시작

- 닉네임, 이미지, 메세지 기능 넣을거임
- Account, Profile 1:1로 매칭 시킬거, 따라서 detail, delete 뷰 따로 구현 안할거

- python manage.py startapp profileApp

- settings.py - INSTALLED_APPS에 'profileApp', 추가

- pinterest - urls.py - urlpatterns에 path('profiles/', include('profileApp.urls')), 추가

```python
#profileApp - urls.py 생성
app_name = 'profileApp'
urlpatterns = [
    
]
```

```python
#profileApp - models.py
class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')	
    image = models.ImageField(upload_to='profile/', null=True)	
    nickname = models.CharField(max_length=20, unique=True, null=True)
    message = models.CharField(max_length=100, null=True)
```

- OneToOneField(): User객체랑 1:1연결
- on_delete=models.CASCADE: 걔 삭제시 얘도 삭제
- related_name: request.user.profile 이런식으로 바로 사용할 수 있도록

- upload_to: media 밑 profile/로 저장
- null=True: 이미지 없어도 됨

- unique: 중복 안됨~

##### ModelForm

- accountApp에서 UserCreationForm은 그냥 장고에서 기본 제공해주는 form이라 그냥 사용했던거고, 다른 앱들 기능은 보통 form 제공 안해줌
- ModelForm은 기존에 있던 모델을 그대로 form으로 변환해주는 기능

```python
#profileApp - forms.py 생성 후
class ProfileCreationForm(ModelForm):
    class Meta:
        model = Profile
        fields = ['image', 'nickname', 'message']
```

##### profileApp - CreateView 구현

- python magage.py makemigrations
- python manage.py migrate 해서 모델 적용 시켜줌

```python
#profileApp - views.py
class ProfileCreateView(CreateView):
    model = Profile
    context_object_name = 'target_profile'
    form_class = ProfileCreationForm
    success_url = reverse_lazy('accountApp:hello_world')
    template_name = 'profileApp/create.html'
```

- profileApp에 templates 폴더 만들고 그 안에 profileApp 폴더 만들어줌
- create.html 만들건데 accountApp-create.html이랑 거의 비슷하므로 복사해서 가져와줌
- 요청 url profileApp으로 바꾸고 화면에 나타나는거 Profile Create로 바꿔줌
- profileApp - urls.py에 urlpatterns에 path('create/',ProfileCreateView.as_view(), name='create')로 추가해줌
- profiles/create로 테스트 해보면 잘 나옴(화면만 잘 나오나 확인)
- 이번엔 create로 갈 수 있는 경로를 만들어줄거임

```html
<!-- accountApp - detail.html 맨 위 </p>아래 부분 수정-->
{% if target_user.profile %}
<h2 style="font-family: 'FONTNAME'">
    {{ target_user.profile.nickname }}
</h2>
{% else %}
<a href="{% url 'profileApp:create' %}">
	<h2 style="font-family: ''">
        Create Profile
    </h2>
</a>
{% endif %}
```

- 이러고 create 테스트 해보면 파일 안넣었다고 오류뜸
- create.html - form 부분에 enctype="multipart/form-data" 추가해줌
- 그러고 다시 테스트해보면 profile.user_id 없다고 오류뜸

```python
#views-py ProfileCreateView() 부분에 메서드 추가
def form_valid(self, form):
    temp_profile = form.save(commit=False)	#DB저장하지 않고 임시로 저장
    temp_profile.user = self.request.user
    temp_profile.save()
    return super().form_valid(form)
```

- 이렇게 하면 해결 됨

##### UpdateView

```python
#views.py에 추가
class ProfileUpdateView(UpdateView):
    model = Profile
    context_object_name = 'target_profile'
    form_class = ProfileCreationForm
    usccess_url = reverse_lazy('accountApp:hello_world')
    template_name = 'profileApp/update.html'
```

- url - urlpatterns에 path 추가, 어떤 프로필에 접근하는지 알야아하니 <u>pk도 추가</u>

```html
<!-- update.html 생성 -->
{% extends 'base.html' %}
{% load bootstrap4 %}

{% block content %}
	<div>
        <div class="mb-4">
            <h4>
                Update Profile
            </h4>
        </div>
        <form action="{% url 'profileApp:update' pk=target_profile.pk %}" method="post" enctype="multipart/form-data">
            {% csrf_token %}
            {% bootstrap_form form %}
            <input type="submit" class="btn btn-dark rounded-pill col-6 mt-3">
        </form>
	</div>
{% endblock %}
```

```html
<!-- detail.html 닉네임 바꾸고플때 update로 라우팅해줄 앵커 태그 삽입 -->
<!-- h2 {{ target_user.profile.nickname }} 아래 추가-->
<a href="{% url 'profileApp:update' pk=target_user.profile.pk %}">
	edit
</a>
```

- 하고 accounts/detail 가보면 edit 뜸 누르면 바꿀 수 있는 곳 들어가짐

- 이번엔 그 이미지 화면에 띄우게끔 만들거임

- detail.html </p> 아래에 <img src=\"{{ target_user.profile.image.url }}\" alt="" style="height: 12rem; width 12rem; border-radius: 20rem; margin-bottom: 2rem;"> 추가

```python
#config - urls.py
from django.conf.urls.static import static
from django.conf import settings
#urlpatterns 끝에 추가
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

- 이러고 /detail 테스트 해보면 잘 나옴
- 이번엔 대화명 나타낼거

```html
<!-- detail.html - {% endif %}아래에 추가 -->
<h5 style="margin-bottom:3rem;">
    {{ target_user.profile.message }}
</h5>
```

##### decorator 설정

```python
#profileApp - decorators.py 생성 후 decorator 설정해줄거임
def profile_ownership_required(func):
    def decorated(request, *args, **kwargs):
        profile = Profile.objeects.get(pk=kwargs['pk'])
        if not profile.user == request.user:
            return HttpResponseForbidden()
        return func(request, *args, **kwargs)
    return decorated
```

- views.py - ProfileUpdateView() 위에 @method_decorator(profile_ownership_required, 'get'), (+'post'도) 추가

##### get_success_url 그리고 리팩토링

- views.py - ProfileCreateView() - success_url 부분 'accountApp: detail' 로 수정 하려함.(이곳으로 가는게 더 알맞음)
- 근데 위처럼 그냥 수정해주면 urls.py - detail부분 보면 pk도 같이 넘겨줘야함 따라서 아래처럼 내부 메소드 수정해줘야함

```python
#views.py - ProfileCreateView(), ProfileUpdateView()에 추가 
def get_success_url(self):
    return reverse('accountApp:detail', kwargs={'pk': self.object.user.pk})
```

- self.object ==> profile 가리킴 거기의 user의 pk를 찾아서 넘겨줌

- detail가서 edit하는거 테스트 해보면 프로필 수정해도 원래대로 돌아옴
- 이번엔 logout해도 edit 나오는거, 다른사람이 들어왔을때 edit 나오는거, 프로필 없는 사람거 들어갔을때 출력해주는거 고쳐줄거임

```html
<!-- detail.html -->
{% extends 'base.html' %}

{% block content %}
    <div>
        <div style="text-align: center; max-width=500px; margin: 4rem; auto;">
            <p>
                {{ target_user.date_joined }}
            </p>
            <img src="{{ target_user.profile.image.url }}" alt="" style="height: 12rem; width: 12rem; border-radius: 20rem; margin-bottom: 2rem;">
            {% if target_user.profile %}
            <h2 style="font-family: 'NanumSquareRoundOTFB'">
                {{ target_user.nickname }}
                    {% if target_user == user %}
                    <a href="{% url 'profileApp:update' pk=target_user.profile.pk %}">
                        edit
                    </a>
                    {% endif %}
            </h2>
            {% else %}
                {% if target_user == user %}
                <a href="{% url 'profileApp:create'%}">
                    <h2 style="font-family: 'NanumSquareRoundOTFB'">
                        Create Profile
                    </h2>
                </a>
                {% else %}
                <h2>
                    닉네임 미설정
                </h2>
                {% endif %}
            {% endif %}
            <h5 style="margin-bottom: 3rem";>
                {{ target_user.profile.message }}
            </h5>
            {% if target_user == user %}
            <a href="{% url 'accountApp:update' pk=target_user.pk %}">
                <p>
                    Change Info
                </p>
            </a>
            <a href="{% url 'accountApp:delete' pk=target_user.pk %}">
                <p>
                    Quit
                </p>
            </a>
            {% endif %}
        </div>
    </div>
{% endblock %}
```

##### MagicGrid 소개 및 ArticleApp 시작

- [MagicGrid](https://github.com/e-oj/Magic-Grid): 게시글 리스팅 해줄 때 핀터레스트를 따올 카드형 레이아웃을 만들어줄 자바스크립트 라이브러리임
- 깃허브 가서 on JSFIDDLE 링크 누르면 실제로 어떻게 되는지 골라볼 수 있음

- python manage.py startapp articleApp
- config-settings 에 'articleApp', 추가
- cofing-url.py에 path 추가

```python
#articleApp - url.py
urlpatterns=[
    path('list/', TemplateView.as_view(template_name='articleApp/list.html'), name='list')
]
```

- articleApp에 templates - articleApp 폴더 만듦

```html
<!-- list.html -->
{% extends 'base.html' %}

{% block content %}
<style>
<!-- css 복붙 -->
</style>
<!-- HTML 복붙 -->
<script src="{% static 'js/magicgrid.js' %}">
</script>
{% endblock %}
```

- 위 코드 위에 {% load static %} 선언해주기
- 추가로  [MagicGrid](https://github.com/e-oj/Magic-Grid) - dist 들어가서 처음 .js 파일 들어가서 내용 복사
- static - js 폴더 만들고 magicgrid.js 만들고 내용 복붙, 마지막 줄 module.exports 이거 지움
- 추가로 magicgrid - script 부분 추가해넣기
- articles/list 테스트 해보면 잘 나옴
- 이제 이거로 커스터마이징 할거임

```html
<!-- list.html -->
{% extends 'base.html' %}
{% load static %}

{% block content %}

<style>
.container div {
  width: 250px;
  background-color: antiquewhite;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 1rem;

  .container img{
    width: 100%;
    border-radius: 1rem;
  }
}

</style>

<div class="container">
    <div class="item1">
        <img src="https://picsum.photos/200/300" alt="">
    </div>
    <div class="item2">2</div>
    <div class="item3">3</div>
    <div class="item4">4</div>
    <div class="item5">5</div>
    <div class="item6">6</div>
    <div class="item7">7</div>
    <div class="item8">8</div>
    <div class="item9">9</div>
    <div class="item10">10</div>
    <div class="item11">11</div>
    <div class="item12">12</div>
    <div class="item13">13</div>
</div>

<script src="{% static 'js/magicgrid.js' %}">
</script>

{% endblock %}
```

- [랜덤으로 사진 가져오는곳](https://picsum.photos/)
- 이러고 테스트하면 자동으로 레이아웃 되지 않음 => 사진 로딩 되는 시간이 있는데 자바스크립트 코드가 끝난 후 이미지 로딩 완료 되어서 그리드 포지셔닝 하는게 반영 안됨

```js
#magicgrid.js 마지막 부분 magicGrid.Listen() 위에 추가
var masonrys = document.getElementsByTagName("img");

for(let i = 0; i < masonrys.length; i++){
    masonrys[i].addEventListener('load', function(){
        magicGrid.positionItems();
    }, false);
}
```

- articles/list 테스트 해보면 잘 정렬 됨

##### ArticleApp 

```python
#articleApp - models.py
class Article(models.Model):
    writer = models.ForeignKey(User, on_delete=models.SET_NULL, related_name='article', null=True)
    title = models.CharField(max_length=200, null=True)
    image = models.ImageField(upload_to='article/', null=False)
    content = models.TextField(null=True)
    
    created_at = models.DateField(auto_created=True, null=True)
```

- SET_NULL로 글 쓴사람이 아이디 삭제해도 남아있게
- auto_created=True 자동으로 생성 됐을 때 값(시간) 설정되게

```python
#articleApp - forms.py
class AritcleCreationForm(ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'image', 'content']
```

- python manage.py makemigrations
- python manage.py migrate

```python
#article - views.py
@method_decorator(login_required, 'get')
@method_decorator(login_required, 'post')
class ArticleCreateView(CreateView):
    model = Article
    form_class = ArticleCreationForm
    template_name = 'articleApp/create.html'
    def form_valid(self, form):
        temp_article = form.save(commit=False)
        temp_article.writer = self.request.user
        temp_article.save()
        return super().form_valid(form)
    def get_success_url(self):
        return reverse('articleApp:detail', kwargs={'pk': self.object.pk})
  
class ArticleDetailView(DetailView):
    model = Article
    context_object_name = 'target_article'
    template_name = 'articleApp/detail.html'
```

- articleApp - templates - articleApp - create.html 전에 만든 create.html 복붙해서 SignUp을 Article Create로 바꿔주고
- \<form action="{% url 'articleApp:create' %}" method="post" enctype='multipart/form-data'\>

```html
<!-- detail.html 만들어서 -->
{% extends 'base.html' %}

{% block content %}
<div>
    <div style="text-align:center; max-width:500px; margin:4rem auto">
        <h1>
            {{ target_article.title }}
        </h1>
        <img src="{{ target_article.image.url }}" alt="">
        
        <p>
            {{ target_article.content }}
        </p>
    </div>
</div>
```

```html
<!-- pinterest - templates - header.html 부분에 nav1 부분 수정 -->
<a href="{% url 'articleApp:list' %}">
	<span>Articles</span>
</a>
```

```html
<!-- list.html - <scrrpt> 윗 부분에 추가 -->
<div style="text-align: center">
    <a href="{% url 'articleApp:create' %}" class="btn btn-dark rounded-pill col-3 mt-3 mb-3">
    	Create Article
    </a>
</div>
```

```python
#articleApp - urls.py
app_name = 'articleApp'
urlpatterns =[
    path('list/', TemplateView.as_view(template_name='articleApp/list.html'), name='list'),
    path('create/', ArticleCreateView.as_view(), name='create'),
    path('detail/<int:pk>', ArticleDetailView.as_view(), name='detail'),
    path('update/<int:pk>', ArticleUpdateView.as_view(), name='update'),
]
```

- articles/list 테스트 해보면 잘 나옴

```python
#articles - views.py에 추가
@method_decorator(article_ownership_required, 'get')
@method_decorator(article_ownership_required, 'post')
class ArticleUpdateView(UpdateView):
    model = Article
    context_object_name = 'target_article'
    form_class = ArticleCreationForm
    template_name = 'articleApp/update.html'
    
    def get_success_url(self):
        return reverse('articleApp:detail', kwargs={'pk': self.object.pk})
```

```python
#articles - decorators.py
def article_ownership_required(func):
    def decorated(request, *args, **kwargs):
        article = Article.objects.get(pk=kwargs['pk'])
        if not article.writer == request.user:
            return HttpResponseForbidden()
        return func(request, *args, **kwargs)
    return decorated
```

```html
<!-- detail.html 맨 아래에 나타나도록 하는 위치에 -->
<a href="{% url 'articleApp:update' pk=target_article.pk %}">
	<p>
        Update Article
    </p>
</a>
```

- 이러고 urls.py에 path('update/\<int:pk\>', ArticleUpdateView.as_view(), name='update'), 추가

```html
<!-- update.html 생성 후 -->
{% extends 'base.html' %}
{% load bootstrap4 %}
{% block content %}
<div>
    <div style="text-align:center; max-width:500px; margin:4rem auto">
        <h4>
            Article Update
        </h4>
    </div>
    <form action="{% url 'articleApp:update' pk=target_article.pk %}" method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {% bootstrap_form form %}
        <input type="submit" class="btn btn-dark rounded-pill col-6 mt-3">
    </form>
</div>
{% endblock %}
```

- 이러고 업데이트 해주는거 테스트(article/detail 에서 Update Article 누르고 진행) 해보면 잘 수정되는거 볼 수 있음

- 이번엔 deleteView 만들거임

```python
#views.py 에 추가
@method_decorator(article_ownership_required, 'get')
@method_decorator(article_ownership_required, 'post')
class ArticleDeleteView(DeleteView):
    model = Article
    context_object_name = 'target_article'
    success_url = reverse_lazy('articleApp:list')
    template_name = 'articleApp/delete.html'
```

```html
<!-- delete.html 생성 후 -->
{% extends 'base.html' %}

{% block content %}

<div style="text-align: center; max-width: 500px; margin: 4rem auto">
    <div class="mb-4">
        <h4>
            Delete Article: {{ target_article.title }}
        </h4>
    </div>
    <form action="{% url 'accountApp:delete' pk=target_user.pk %}" method="post">
        {% csrf_token %}
        <input type="submit" class="btn btn-danger rounded-pill col-6 mt-3">
    </form>
</div>

{% endblock %}
```

```html
<!-- detail.html 맨 아래 나오게끔 추가 -->
<a href="{% url 'articleApp:delete' pk=target_article.pk %}">
	<p>
        Delete Article
    </p>
</a>
```

- url.py 에 path('delete/\<int:pk\>', ArticleDeleteView.as_view(), name='delete'), 추가

- 이렇게해서 삭제해보는거 test 해보기~

- 잘되면 .gitignore에 media/ 추가해서 커밋~

##### ListView, Pagination 적용

- Pagination: 페이지의 객체를 생성한다라고 생각하면 됨 ex) google-1,2,3,4 이렇게 페이지 형식
- cf) Infinite Scroll: 무한 스크롤 ex) facebook

```python
#articleApp - views.py에 추가
class ArticleListView(ListView):
    model = Article
    context_object_name = 'article_list'
    template_name = 'articleApp/list.html'
    paginate_by = 25
```

```python
#urls.py - urlpatterns에서 수정(있던거 지워야함)
path('list/' ArticleListView.as_view(), name='list'),
```

```html
<!-- list.html </style> 아래 수정 -->
{% if article_list %}
<div class="container">
    {% for article in article_list %}
    <a href="{% url 'articleApp:detail' pk=article.pk %}">
    	{% include 'snippets/card.html' with article=article %}
    </a>
    {% endfor %}
</div>
<script src="{% static 'js/magicgrid.js' %}"></script>
{% else %}
<div class="text-center">
    <h1>
        No Articles YET!
    </h1>
</div>
{% endif %}
<div style="text-align: center">
    <a href="{% url 'articleApp:create' %}" class='btn btn-dark rounded-pill col-3 mt-3 mb-3 '>
    	Create Article
    </a>
</div>
```

- with 구문 사용해서 card.html에 쓰일 article이 위에 for 문으로 돌리는 article과 같다는 뜻

```html
<!-- article - templates - snippets 폴더 생성 후 card.html 생성 -->
<div>
    <img src="{{ article.image.url }}" alt="">
</div>
```

- 이러고 articles/list 테스트 해보고, 게시글 만들어서 테스트 해보기, 또 글 많이 넣어가지고 페이지네이션 잘되나 확인해보기(views.py - ArticleListView() - paginate_by 수 줄이면 좀 더 쉽게 가능)

- 이제 그 숫자로 페이지 옮길 수 있게 만들어줄거

- list.html {% endif %} 다음에 {% include 'snippets/pagination.html' with page_obj=page_obj %} 추가

```html
<!-- snippets - pagination.html 생성 후--> 
<div style="text-align: center; margin: 1rem 0;">
    {% if page_obj.has_previous %}
    <a href="{% url 'articleApp:list' %}?page={{ page_obj.previous_page_number }}" class="btn btn-secondary rounded-pill">
    	{{ page_obj.previous_page_number}}
    </a>
    {% endif %}
    <a href="{% url 'articleApp:list' %}?page={{ page_obj.number }}" class="btn btn-secondary rounded-pill active">
        {{ page_obj.number}}
    </a>
    {% if page_obj.has_next %}
    <a href="{% url 'articleApp:list' %}?page={{ page_obj.next_page_number }}" class="btn btn-secondary rounded-pill">
    	{{ page_obj.next_page_number}}
    </a>
    {% endif %}
</div>
```

- page_obj.has_previous: 이전페이지가 있는지
- 있으면 이전 페이지로 넘어가는 링크 만들어줌
- {% endif %}다음은 현재 있는 페이지를 향하는 링크
- page_obj.has_next: 다음페이지가 있는지
- 있으면 다음페이지로 넘어가는 링크(숫자) 만들어줌

- 이러고 페이지 번호 눌러가며 테스트(articles/list) 해보면 잘 됨
- 다음은 코멘트앱 만들거, 디테일뷰 제일 아래 만들거라 디자인 정리좀 해줄거

```html
<!-- detail.html 수정 -->
{% extends 'base.html' %}

{% block content %}
<div>
    <div style="text-align:center; max-width:700px; margin:4rem auto">
        <h1>
            {{ target_article.title }}
        </h1>
        <h5>
            {{ target_article.writer.profile.nickname }}
        </h5>
        <hr>

        <img style="width: 100%; border-radius: 1rem; margin: 2rem 0;"
             src="{{ target_article.image.url }}" alt="">

        <p>
            {{ target_article.content }}
        </p>

        {% if target_article.writer == user %}
        <a href="{% url 'articleApp:update' pk=target_article.pk %}" class="btn btn-primary rounded-pill col-3 ">
            Update
         </a>
        <a href="{% url 'articleApp:delete' pk=target_article.pk %}" class="btn btn-danger rounded-pill col-3 ">
            Delete
        </a>
        {% endif %}
        <hr>

    </div>
</div>
{% endblock %}
```

- list 테스트 해보기~

##### Mixin을 이용한 게시글 댓글 시스템(CommentApp) 구현

- Mixin: 다중상속 받아서 DetailView 안에서도 form을 사용하게끔(addon 느낌? 이래)
- python manage.py startapp commentApp
- config - settings.py에 APP 추가, urls.py에 path 추가

```python
#commentApp - models.py
class Comment(models.Model):
    article = models.FroeignKey(Article, on_delete=models.SET_NULL, null=True, related_name='comment')
    writer = models.FroeignKey(User, on_delete=models.SET_NULL, null=True, related_name='comment')
    content = models.TextField(null=False)
    created_at = models.DateTimeField(auto_now=True)
```

- article, writer 서버단에서 확인할거
- content 입력 받을거
- created_at은 자동으로 생성될거
- model 만들었으면 migration 작업 해주기(이걸 forms.py까지 완성하고 해야는지 여기서 해도 되는지 해보기)

```python
#commentApp - forms.py 생성
class CommentCreationForm(ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
```

```python
#commentApp - views.py
class CommentCreateView(CreateView):
    model = Comment
    form_class = CommentCreationForm
    template_name = 'commentApp/create.html'
    def get_success_url(self):
        return reverse('articleApp:detail', kwargs={'pk': self.object.article.pk})
```

```python
#commentApp - urls.py 생성
app_name = 'commentApp'
urlpatterns = [
    path('create/', CommentCreateView.as_view(), name='create'),
]
```

- templates - commentApp 폴더 생성하고 create.html 생성

```html
<!-- create.html -->
{% extends 'base.html' %}
{% load bootstrap4 %}

{% block content %}

	<div style="text-align: center; max-width: 500px; margin: 4rem auto">
        <div class="mb-4">
            <h4>
                SignUp
            </h4>
        </div>
        <form action="{% url 'commentApp:create' %}" method="post">
            {% csrf_token %}
            {% bootstrap_form form %}
            <!-- 로그인 되어있을때만 버튼 보여줄거임 -->
            {% if user.is_authenticated %}
            <input type="submit" class="btn btn-dark rounded-pill col-6 mt-3">
            {% else %}
            <a href="{% url 'accountApp:login' %}?next={{ request.path }}" class = "btn btn-dark rounded-pill col-6 mt-3">
                Login
            </a>
            {% endif %}
            
			<input type="hidden" name="article_pk" value="">
        </form>
	</div>

{% endblock %}
```

- 이러고 comments/create 테스트 해보면 잘 나옴

- 이제 만든 레이아웃을 게시글 아래쪽으로 배치할거임

- articleApp-templates-articleApp-detail.html 가서 맨 아래쪽 나오게 {% include 'commentApp/create.html' with article=target_article %} 이래주면 create안에서 article이라는 변수 사용 가능하게됨

- create.html \<input type="hidden" name="article_pk" value="{{ <u>article.pk</u> }}"\>로 수정
- ~~create.html 맨 위 {% extends 'base.html' %} 지워도 됨, 두번 extends하게 되는거래~~
- 이러고 테스트 해보면 form 문제 있음 이제 mixin사용해줘야함
- articleApp - views.py - ArticleDetailView()에서 FormMixin 인자로 추가해줘서 다중상속 시켜줌, 내용에 form_class = CommentCreationForm 추가해줌
- 이러고 articles/detail/ 테스트 해보면 잘 작동함
- 근데 이대로 comment 달 순 없음 원하는대로 작동시키려면 내용 더 추가

```python
#commentApp - views.py - CommentCreateView()에 내용 추가
def form_valid(self, form):
    temp_comment = form.save(commit=False)
    temp_comment.article = Article.objects.get(pk=self.request.POST['article_pk'])
    temp_comment.writer = self.request.user
    temp_comment.save()
    return super().form_valid(form)
```

- 이러고 테스트 해보면 잘 됨 근데 댓글들 나타나지 않음 이제 그거 해결해줄거

```html
<!-- articleApp - detail.html 마지막 {%include 구문} 위에 추가-->
{% for comment in target_article.comment.all %}
	{% include 'commentApp/detail.html' with comment=comment %}
{% endfor %}
```

```html
<!-- commentApp - detail.html 생성 -->
<div style="border: 1px solid; text-align: left; padding: 4%; margin: 1rem 0; border-radius: 1rem; border-color: #bbb">
    <div>
		<strong>
        	{{ comment.writer.profile.nickname }}
        </strong>
        &nbsp&nbsp&nbsp		<!-- 띄어쓰기 -->
        {{ comment.created_at }}
    </div>
    <div style="margin: 1rem 0;">
        {{ comment.content }}
    </div>
</div>
```

- 이러고 테스트 해보면 잘 달림
- 이제 삭제하는거 DeleteView 만들거

```python
#commentApp - views.py에 추가
class CommentDeleteView(DeleteView):
    model = Comment
    context_object_name = 'target_comment'
    template_name = 'commentApp/delete.html'
    def get_success_url(self):
        return reverse('articleApp:detail', kwargs={'pk': self.object.article.pk})
```

```html
<!-- detail.html 마지막에 나오게끔 추가 -->
{% if comment.writer == user %}
<div style="text-align: right">
    <a href="{% url 'commentApp:delete' pk=comment.pk %}" class="btn btn-danger rounded-pill">
    	Delete
    </a>
</div>
{% endif %}
```

- urls.py에 path('delete/<int:pk>', CommentDeleteView.as_view(), name='delete'), 추가

```html
<!-- commentApp - delete.html 생성 -->
{% extends 'base.html' %}

{% block content %}

<div style="text-align: center; max-width: 500px; margin: 4rem auto">
    <div class="mb-4">
        <h4>
            Delete Comment: {{ target_comment.content }}
        </h4>
    </div>
    <form action="{% url 'commentApp:delete' pk=target_comment.pk %}" method="post">
        {% csrf_token %}
        <input type="submit" class="btn btn-danger rounded-pill col-6 mt-3">
    </form>
</div>

{% endblock %}
```

- 이러고 삭제 테스트 해보면 잘 됨

- 추가적으로 comment 주인인지 확인하는거

```python
#commentApp - decorators.py 생성
def comment_ownership_required(func):
    def decorated(request, *args, **kwargs):
        comment = Comment.objects.get(pk=kwargs['pk'])
        if not comment.writer == request.user:
            return HttpResponseForbidden()
        return func(request, *args, **kwargs)
    return decorated
```

- commentApp - views.py - CommentDeleteView 위에 @method_decorator(comment_ownership_required, 'get') 추가(post도) 

- 이러고 테스트 해보면 로그아웃 했을때 delete 버튼 안보임

##### 모바일 디버깅, 반응형 레이아웃

- python manage.py runserver 0.0.0.0:8000 으로 실행하면 컴퓨터 말고도 다른 곳에서 ip주소를 기반으로 서버에 접속할 수 있는 포트가 열림
- 모바일로 ip주소:8000/accounts/login 접속해보면 호스트 허용되지 않았다고 에러뜸
- config - settings.py - ALLOWED_HOSTS = ['*'] 이래주면 모든 호스트 허용됨

- 이러고 모바일로 들어가보면 모바일 최적화 안되어있음
- 이걸 실제 디바이스 너비에 맞춰서 리사이즈 해주는 태그를 html안에 넣을거임

```html
<!-- templates - head.html - <head> - <meta> 아랫줄에 추가-->
<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
```

- shrink~: firefox에 맞추는 셋팅
- initia~: 처음 스케일이 1
- width=device~: 디바이스 너비에 맞춰서 간격 맞춰줌
- 이러고 모바일로 테스트 해보면 아까보단 나아졌는데 모바일에 신경 안쓰고 짠거라 좀 더 보완

```html
<!-- articleApp - list.html - <style> 맨 위에 추가 -->
background-color, width: 250px 지우고
.container{
    padding: 0;
    margin: 0, auto;
}

.container a{
	<!-- 이런식으로 하면 보통 모바일에선 45%되고 데스크탑에선 250px로 맞춰짐 -->
	width: 45%;
	max-width: 250px;
}
```

- 이러고 테스트 해보면 잘 됨, 이번엔 간격 다닥다닥 붙도록 해줄거

```js
/* static - js - magicGrid.js - let megicGrid = new MagicGrid() 부분 수정 */
gutter: 12,
```

- create Article 부분이 좀 엉성한데 수정해주면,
- articleApp - list.html - create Article 부분 class에 px-3 추가

- 모바일에서 크기가 살짝 큰 느낌일때 좀 줄이는 방법

```css
/* static - base.css 마지막에 추가 */
@media screen and (max-width:500px){
    html {
        font-size: 13px;
    }    
}
```

- 스크린 크기가 500 이하로 떨어지면 이 안에 있는 css 태그들이 적용 된다는 뜻

- 데스크탑으로 켜서 f12 - 모바일 모드로 하고 너비 줄여보면 재조정 되는거 볼 수 있음

##### projectApp(게시판) 구현 - 좀 설명하기 복잡한거 git - commit으로 표시하기

- 나머지 비슷하다고 알아서 만들래 수정할 부분만 알려준대
- 40강 보면서 차근차근 만들어도 될듯, 빠르게라도 다 설명해주긴 함
- enctype: 이미지 들어가는 곳이면 넣어줌
- \'object-fit: cover\':높이-너비 설정해줬는데, 비율 다른거 찌그러지지 않게 자동으로 잘라주는 옵션
- pagination에 url이 articleApp으로 고정되어 있는데 이거 제거하게 되면 어떤 페이지에서 이 페이지네이션을 불러오더라도 그 앞에 있는 url은 그대로 있는데 뒤에 있는 get parameter만 바뀌어서 가기 때문에 어디서든 사용할 수 있게 됨

- {{ project.title | truncatechars:8 }} 이런식으로 하면 원하는 글자만큼 잘라서 나타내줌
- base.css에서 태그별로 설정해줄 수 있음
- \'a:hover\' 이건 마우스 올렸을 때 어떤 이벤트를 발생할지
- style - class 이거 base.css에 보면 매칭되는게 있음 여기서 고쳐주는거

##### MultipleObjectMixin을 통한 projectApp 마무리

- Project 안에 article들이 들어가게끔 여기서 연결 해줄거

```python
#articleApp - models.py - Article() - writer 아래 추가
project = models.ForeignKey(Project, on_delete=models.SET_NULL, related_name='article', null=True)
```

```python
#articleApp - forms.py - ArticleCreationForm() 수정
fields = ['title', 'image', 'project', 'content']
```

- model 변경했으니 makemigrations, migrate 해줌
- articles/create 테스트 해보면 프로젝트 선택 가능

```python
#projectApp - views.py - ProjectDetailView()에 추가
#인자에 MultipleObjectMixin 추가
paginate_by = 25
def get_context_data(self, **kwargs):	#실질적으로 어떤 게시물 가져올지 필터링 하는곳
    object_list = Article.objects.filter(project=self.get_object())
    return super(ProjectDetailView, self).get_context_data(object_list=object_list, **kwargs)
```

- articleApp - list.html 복사해서 templates-snippets-list_fragment.html 만들어서 붙여넣기
- extends, block 구문 지워줌

```html
<!-- projectApp - detail.html 맨아래 </div>위에 추가 -->
<div>
    {% include 'snippets/list_fragment.html' with article_list=object_list %}
</div>
```

- 이러고 projects/detail 테스트 해보면 만든 게시글 잘 나와있음
- 이번엔 accounts/detail 아래쪽에도 나오게끔 똑같이 적용
- accountApp - views.py에 projectApp - views.py - ProjectDetailView() 에 한 형식대로(바로 위위 코드)
- accountApp - detail.html 가서 \<div\> 똑같이(바로 위 코드)
- 이러고 accounts/detail 테스트 해보면 내가 쓴글들 잘 나옴

##### RedirectView를 통한 subscriptionApp(구독 기능) 시작

- python magage.py startapp subscriptionApp
- settings.py APP, urls.py path 추가

- subscriptionApp - urls.py 생성

```python
#subscriptionApp - models.py
class Subscription(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='subscription')
    project = models.ForeignKey(Project, on_delete=models.CASCADE, related_name="subscription")
    
    #user - project 쌍이 가지는 구독 정보가 하나여야하니까(한 유저 같은거 구독 한번만)
    class Meta:
        unique_together = ('user', 'project')
```

- makemigration, migrate 해주기

```python
#views.py
@method_decorator(login_required, 'get')
class SubscriptionView(RedirectView):
    
    def get_redirect_url(self, *args, **kwargs):
        return reverse('projectsApp:detail', kwargs={'pk': self.request.GET.get('project_pk')})
    def get(self, request, *args, **kwargs):
        project = get_object_or_404(Project, pk=self.request.GET.get('project_pk'))
        user = self.request.user
        subscription = Subscription.objects.filter(user=user, project=project)
        if subscription.exists():	#있으면 구독 취소
            subscription.delete()
        else:						#없으면 구독
            Subscription(user=user, project=project).save()
        return super(SubscriptionView, self).get(request, *args, **kwargs)
```

- get_redirect_url() 내용: project_pk를 get방식으로 받아서 이 pk를 가지고 있는 detail 페이지로 되돌아간다는 내용
- get()-project 내용: project_pk를 가지고 있는 Project를 찾는데, 없다면 404 뱉음
- get()-subscription: 위에 두가지 정보를 가지고 subscription을 찾음
- 이번엔 이 일을 해줄 버튼을 만들어줄거임

```html
<!-- projectApp - detail.html 게시글 보여주는거 위쪽에 추가 -->
<div class="text-center mb-5">
    {% if user.is_authenticated %}
    <a href="{% url 'subscriptionApp:subscription' %}?project_pk={{ target_project.pk }}" class="btn btn-primary rounded-pill px-4">
    	Subscription
    </a>
    {% endif %}
</div>
```

- is_authenticated: 로그인 한사람한테만 보이도록 하는거

- subscriptionApp - urls.py에 path('subscription/', SubscriptionView.as_view(), name='subscription') 추가

- tip) margin 4개로 쓰면 위부터 시계방향으로 설정됨

- 이러고 테스트해보면 뭔가 되는거 같은데 바뀌는거 없음 버튼이 어떻게 행동할지 수정해줄거

```python
#projectApp - views.py - ProjectDetailView() - get_context_data()에 추가
project = self.object
user = self.request.user
if user.is_authenticated:	#접속 되어있으면
    subscription = Subscription.objects.filter(user=user, project=project)
#return get~ 부분 수정
get_context_data(object_list=object_list, subscription=subscription, **kwargs)
```

```html
<!-- projectApp - detail.html 게시글 보여주는거 위쪽 부분 수정 -->
<div class="text-center mb-5">
    {% if user.is_authenticated %}
    	{% if not subscription %}
        <a href="{% url 'subscriptionApp:subscription' %}?project_pk={{ target_project.pk }}" class="btn btn-primary rounded-pill px-4">
            Subscription
        </a>
    	{% else %}
    	<a href="{% url 'subscriptionApp:subscription' %}?project_pk={{ target_project.pk }}" class="btn btn-dark rounded-pill px-4">
            UnSubscription
        </a>
    	{% endif %}
    {% endif %}
</div>
```

- 이러고 테스트 해보기(projects/detail)

##### field lookup을 사용한 구독 페이지 구현

![img](https://blog.kakaocdn.net/dn/WLeDQ/btqRlIa2xum/z180YAYm2Hl86I0Cw8wWC1/img.png)

- 목적은 좀 더 복잡한 DB 쿼리를 사용자가 구현할 수 있도록 만들어 주는것
- 좀 더 자세한 내용은 [여기](https://docs.djangoproject.com/en/3.1/ref/models/querysets/#field-lookups)

```python
#subscriptionApp - views.py 에 추가
@method_decorator(login_required, 'get')
class SubscriptionListView(ListView):
    model = Article
    context_object_name = 'article_list'
    template_name = 'subscriptionApp/list.html'
    paginate_by = 5
    
    def get_queryset(self):
        projects = Subscription.objects.filter(user=self.request.user).vlaues_list('project')
        article_list = Article.objects.filter(project__in=projects)
        return article_list
```

- values_list => 값들을 list화 시킴

```html
<!-- subscriptionApp - templates - subscriptionApp 폴더 만들고 list.html 생성-->
{% extends 'base.html' %}
{% block content %}

	<div>
        {% include 'snippets/list_fragment.html' with article_list=article_list %}
	</div>
{% endblock %}
```

- urls.py에 path('list/' SubscriptionListView.as_view(), name='list'), 추가

```html
<!-- templates - header.html 에서 3번째 Subscription 탭 만들도록 추가해줌 -->
<a href="{% url 'subscriptionApp:list' %}" class="pinterest_header_nav">
	<span>Subscription</span>
</a>
```

- 하고 subscription/list 테스트 해보면 구독 여부에 따라 잘 나옴

##### WYSIWYG(What You See Is What You Get) 적용

- 게시판 기능 중 하나
- [미디엄에디터](https://github.com/yabwe/medium-editor) 적용할거, 여기가서 readme의 demo 부분 링크타보면 어떤식으로 가능한지 볼 수 있음

```python
#articleApp - forms.py - ArticleCreationForm() 내부에 추가
#from django import forms
content = forms.CharField(widget=forms.Textarea(attrs={'class': 'editable'}))
```

```html
<!-- articles - create.html - block content 내부 맨 위에 추가 -->
<!-- 사이트 readme에서 찾아서 복붙한거 -->
<script src="//cdn.jsdelivr.net/npm/medium-editor@5.23.2/dist/js/medium-editor.min.js"></script>
<link rel="stylesheet" href="//cdn.jsdelivr.net/npm/medium-editor@5.23.2/dist/css/medium-editor.min.css" type="text/css" media="screen" charset="utf-8">

<!-- 이 친구는 endblock 위에 추가 -->
<script>var editor = new MediumEditor('.editable');</script>
```

- 이러고 create 테스트하면 글자 바꿀 수 있는데 엔터 여러번 쳐보면 박스가 안따라옴 이거 처리해주려면

- articleApp - forms.py - ArticleCreationForm() attrs={'class': 'editable text-left', 'style': 'height: auto;'}))  ==> class에 text-left 적용 안돼서 그냥 style에 text-align: left 추가해줌

- 컨텐트 필드가 만들어 질때 클래스랑 스타일을 여기서 미리 결정해준다 생각해주면 됨
- 이러고 테스트 해보면 잘 됨, 근데 여기서 글 써보면 글에 태그가 엄청 많이 나타남, 컨텐트를 그대로 뿌려줘서 그럼 이거 고쳐줄거

```html
<!-- articleApp-detail.html의 {{ target_article.content }} 부분 수정 -->
<div class="text-left">					<!-- p => div로 -->
    {{ target_article.content | safe }}
</div>
```

- 이러고 테스트 해보면 잘되는데 update 눌러보면 잘 안나옴 update뷰도 적용해줘야함

- update.html 가서 바로 위위 코드처럼 추가해주고

```python
#articleApp - forms.py - ArticleCreationForm() 내부에 추가
project = forms.ModelChoiceField(queryset=Project.objects.all(), required=False)
```

- required=False 해서 선택 없이도 제출 가능

##### 정리 및 다듬기

- 게시글 쓸 때 프로젝트 고르는거 이름으로 나타나도록

```python
#projectApp - models.py - Project()에 추가
def __str__(self):
    #따옴표 앞에 f 붙이면 이 안에 직접 변수 출력 가능
    return f'{self.pk}: {self.title}'	
```

- 이러고 테스트 해보면 프로젝트 선택시 몇번 글의 이름 으로 나옴

```python
#projectApp - views.py - get_context_data() 부분 수정
#if user 아닌경우 문제 생겨서 아닌경우도 처리해줘야함, 아래 내용 if아래 추가
else: subscription = None
```

- accountApp - hello_world.py 이거 과거의 잔재들 제거할거
- pinterest 우클릭 Find in path 해서 hello치고 in project 누르면 hello 나온거 다 나옴 필요없는 부분들 지우기 (<u>주의! migrations 파일은 지우면 안됨</u>)
- LOGIN_REDIRECT_URL 이 부분은 reverse_lazy('home')으로 바꿔줌
- 이번엔 homepage 설정할거

```python
#config - urls.py - urlpatterns에 추가
path('', ArticleListView.as_view(), name='home'),
```

- 이제 그냥 주소만 쳐도  들어가짐
- 이번엔 edit부분 change info, quit 부분 좀 더 이쁘게 바꿔줄거

- [아이콘](https://material.io/resources/icons/?style=baseline) 이거 이용할거

- 아이콘들 이름들이 나와있는데 이름만 넣고 클래스만 바꿔주면 아이콘 적용 가능
- [아이콘깃헙](https://github.com/google/material-design-icons) 여기 보면 사용법 나와있음
- Using a font 부분 링크 복사해서

```html
<!-- templates - head.html - GOOGLE FONTS LINK 아래 추가 -->
<!-- GOOGLE METERIAL ICONS -->
<link href="https://fonts.googleapis.com/css2?family=Material+Icons"
      rel="stylesheet">
```

```html
<!-- accountApp - detail.html edit 부분 추가 -->
<a class="material-icons" style="box-shadow: 0 0 4px #ccc; border-radius: 10rem; padding: .4rem;"
```

- box-shadow: (x, y, 크기, 색)

```html
<!-- 위에거 복사해서 Chang Info, Quit 부분에 복붙하고 사용하고픈 이름 사용-->
<!-- ex) Chang Info => settings, Quit => cancel -->
<!-- cancel 부분 색 그림자 붉은색으로 #fcc -->
```

- 마지막으로 config - settings.py - STATIC_URL 부분 = '/static/' 으로 바꿔주기, docker 볼륨 쓸 때 오류남

---

- articles/detail 하면 header 부분이 두개 나와 이거 고쳐야함(공부하며 작동 방식 이해하고 고치기)

- 로그인 안하고 subscription 누르니까 에러남