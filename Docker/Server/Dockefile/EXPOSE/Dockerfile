#Doickerfile_HTTPD3 파일명으로 다음과 같이 작성
FROM ubuntu:18.04
LABEL maintainer="asdf134652@gmail.com"
LABEL version="1.0.0"
LABEL description="A test Docekr image to understanding"       

RUN apt-get update #패키지(프로그램) 정보 업데이트
RUN apt-get install -y apache2 apt-utils #apache2 패키지(프로그램) 설치, 중간에 Y/N 묻는 단계가 나오면 모두 Yes로 하고 설치

EXPOSE 80

#apache2 디폴트 웹서버 설정은 /var/www/html/ 폴더의 웹페이지를 보여줌
COPY ./2021_DEV_HTML /var/www/html/

#apache2 웹서버 구동 명령은 다음과 같음 
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]