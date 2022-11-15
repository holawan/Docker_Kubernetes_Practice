# Dockerfile_history 
FROM ubuntu:18.04
LABEL maintainer='holawan'
LABEL version="1.0.0"
LABEL description="test"

RUN apt-get update #패키지(프로그램) 정보 업데이트
RUN apt-get install -y apache2 #apache2 패키지(프로그램 )설치, 중간에 Y/N 묻는 단계가 나오면 모두 Yes로 하고 설치

#apache2 디폴트 웹 서버 설정은 /var/www/html/폴더의 웹 페이지를 보여줌

COPY ./2021_DEV_HTML /var/www/html

ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
