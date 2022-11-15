## python 서버 구축을 위한 웹서버 이해

### Flask webserver

- nginx proxy로 reverse proxy를 구성하고
- HTML/CSS/JS 지원을 위한 내부 nginx 서버와,
- 워드프레스 지원을 위한 워드프레서 서버와(나의 도메인/blog/경로로 설정)
- flask 백엔드 지원과, flask 성능 개선을 위한 gunicorn 프로그램을 활용한 서버로 구성 (나의 도메인/util/ 경로로 설정하되, 내부에서는 /util 경로를 삭제한 나의 도메인으로 경로가 변경되도록 설정)

### Web Server와 WSGI

> 우선 구현 전에 기본적인 백그라운드 지식을 이해하면, 훨씬 좋은 개발자가 될 수 있습니다.

#### Webserver와 WAS 프레임워크

- 웹서버는 정적인 HTML페이지를 반환한다.
  - 요청에 따른 정적인 데이터를 반환한다.
  - 주요 웹서버로는 익히 알려진 Apache와 nginx가 있음
- 웹서버가 동적으로 데이터를 반환하도록 하기 위해서는, WAS 프레임워크가 필요하다.
  - 주요 WAS 프레임워크로는 flask, django,rails, node.js 등이 있다.

#### WAS 프레임워크

- 웹 서버 위에서 동작하는 서버 응용프로그램을 WAS라고 부름
- WAS를 개발하기 위한 프레임워크를 사용해서, WAS를 개발할 수 있음

#### WSGI

- Web server Gateway Interface
- 파이썬 스크립트(응용프로그램)이 웹서버와 통신할 수 있도록 지원하는 프로토콜
  - WSGI는 파이썬을 위한 기능 
- 주요 기능
  - 웹서버의 환경 정보와 콜백함수를 파이썬 스크립트에 전달해주는 기능
  - 앱에서 요청을 처리한 결과를 콜백함수를 통해 웹서버에 전달해주는 기능 

#### Middleware

- WSGI의 동작을 지원하는 프로그램
- 주요 기능
  - Middleware가 응용 프로그램을 실행시켜줌(응용 프로그램 컨테이너)
  - 웹 서버의 환경 정보 변경 시 이에 따른 요청 경로 변경
  - 동일 프로세스 상에서 여러 응용프로그램 실행 지원
  - XSLT 스타일시트 적용 등 전처리 지원
- 주요  Middleware
  - mod_wsgi,uwsgi,gunicorn,twisted.web, tornado 등

### 정리

- 결국, 파이썬 응용 프로그램을 웹 서버 상단에서 실행시킬 수 있도록 하기 위해, WSGI라는 프로토콜을 만들고, 여기에 편의성을 더한 기능을 추가한 WSGI 미들웨어가 존재하는 것

- 웹서버와 WSGI 동작 방식

  1. 웹서버가 설정에 따라 특정 포트를 listening
  2. 요청이 들어오면 웹서버가 쓰레드를 생성해서 해당 요청 처리
     - 해당 쓰레드는 WSGI Middleware를 통해 파이썬 응용프로그램에 요청을 전달
     - 파이썬 응용 프로그램이 요청을 처리해서 다시 WSGI Middleware에 전달
  3. WSGI Middleware가 웹서버 쓰레드에 해당 요청 처리 결과를 전달

  > 우리는 nginx proxy가 웹서버가 되고, 웹서버가 gunicorn WSGI Middleware 프로그램에 해당 request 처리를 요청하도록 구성

- mod_wsgi : apache의 플러그인(모듈)로 구현된 WSGI Middleware
- uwsgi : C언어로 작성된 WSGI Middleware
- gunicorn : python 언어로 작성된 WSGI Middleware

> 기존에는 uwsgi를 많이 사용했지만, uwsgi는 무겁고 resource를 많이 쓰는 경향이 있음
>
> 따라서 gunicorn을 사용하기로 함

