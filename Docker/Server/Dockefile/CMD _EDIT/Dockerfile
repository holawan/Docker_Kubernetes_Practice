#Doickerfile_HTTPD3 파일명으로 다음과 같이 작성
FROM httpd:alpine
LABEL maintainer="asdf134652@gmail.com"
LABEL version="1.0.0"
LABEL description="A test Docekr image to understanding"       

COPY ./2021_DEV_HTML /usr/local/apache2/htdocs

ENTRYPOINT ["/bin/echo","hello"]