# HTTPS 지원 서비스 구축

## HTTPS

- 최근 검색엔진은 https를 지원하는 사이트를 우선 지원함
  - 예 : 구글 검색 엔진은 https를 지원하는 사이트를 검색시 상위에 올려줌 
- https를 지원하기 위해서는 인증서 발급이 필요하며, 보통 연단위 비용이 청구됨



### Let's Encrypt와 certbot

- 연단위 비용없이 인증서를 발급해주는 서비스
- 단, 일정 기간마다(90일) 갱신해줘야 함 

### Let's Encrypt SSL 인증서 발급 명령 예

- Certbot 프로그램을 제공해주며, 해당 프로그램에 옵션을 넣어서, 인증서를 발급받을 수 있음

```
certbot certonly --cert-name cert --standalone -d domain.com,www.domain.com
```



### HTTPS 테스트를 위한 사전 준비

> 여기서부터는 도메인이 필요하고 도메인과 고정 IP가 설정한 서버가 필요하며 AWS 가입 및 EC2 서버 구축과 도메인 구입이 필요함
>
> 리눅스 사용법과 AWS 사용법은 또다른 별도 강의 분량이므로, 관련 내용에 대해 별도 강의 또는 책으로 익힌 후, 테스트

#### 도메인 구입 및 DNS 레코드 설정

- 자신의 EC2 IP를 도메인에 연결
- 이를 위해 네임서버가 필요하며, 가비아에서 도메인 구입시 네임서버를 지원하며, 가비아에서 DNS 레코드 설정 필요
- 참고 : https://customer.gabia.com/manual/dns/3041/3040



#### certbot과 nginx 기본 설정

- certbot 옵션 이해

```
certonly --dry-run --webroot --webroot-path=/usr/share/nginx/html --email asdf134652@gmail.com --agree-tos --no-eff-email --keep-until-expiring -d familyzoa.com -d www.familyzoa.com
```

- certbot certonly : certonly는 certbot 프로그램의 플러그인으로 인증서를 받는 서브명령
  - certbot/certbot 이미지는 entrypoint가 certbot이기 때문에, command에서는 certonly부터 작성함
  - certonly는 새로 인증서를 받거나, 갱신된 인증서를 받기만 하며, certonly와 --webroot 옵션을 함께 써서, certbot 프로그램이 알맞은 위치에 인증서를 설치하게 됨
- **--dry-run : 테스트에서는 반드시 이 옵션을 써서 테스트, 이 옵션 없이 수차례 테스트하면, 에러 발생**
- --webroot : 내 서버에 인증 정보를 넣고, 이를 기반으로 인증서를 발급받겠다는 옵션(인증서 갱신을위해, 웹 서버를 끄지 않아도 됨)
- --webroot-path : 인증 정보를 넣을 때 서버의 기본 폴더를 지정
- --email : 인증서 복구 등을 위한 이메일 등록
- --agree-tos : ACM 서버 구독 동의
- --no-eff-eamil : 해당 이메일 주소를 certbot 회사에 공유하지는 않음
- --keep-until-expiring : 아직 인증서가 renew가 필요하지 않으면, 갱신하지 않고 기존 인증서를 보존함



#### docker-compose.yml

```
version: "3"

services:
  webserver:
    image: nginx:latest
    container_name: proxy 
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./myweb:/usr/share/nginx/html
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./certbot-etc:/etc/letsencrypt

  nginx:
    image: nginx:latest
    container_name: myweb
    restart: always
    volumes:
      - ./myweb:/usr/share/nginx/html

  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./certbot-etc:/etc/letsencrypt
      - ./myweb:/usr/share/nginx/html
    command: certonly --dry-run --webroot --webroot-path=/usr/share/nginx/html --email asdf134652@gmail.com --agree-tos --no-eff-email --keep-until-expiring -d familyzoa.com -d www.familyzoa.com

```

#### 인증서 발급 결과 확인

```
$ docker logs certbot
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Account registered.
Simulating a certificate request for familyzoa.com and www.familyzoa.com
The dry run was successful.
```



### Reverse Proxy 구축

#### 인증서 파일 확인 (dry-run 옵션 삭제 후, 실제 인증서 발급시)

```
#EC2 root 사용자 패스워드 설정
sudo passwd

# root 사용자로 사용자 전환
su - 

cd /home/ubuntu/08_HTTPS/certbot-etc/live
ls
#자신의 도메인 폴더가 있고 ,해당 폴더 안에 인증서가 있음 
```

#### 보안 옵션 개선

- 호스트 PC에서 docker compose에 들어갈 volume에 직접 다음 두 파일을 정확한 위치에 다운로드 받음
- curl 다운로드 및 설치 (https://curl.se/download.html 또는 https://winampplugins.co.uk/curl/)
  - options-ssl-nginx.conf : https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf
  - ssl-dhparams.pem : https://raw.githubusercontent.com/certbot/certbot/master/certbot/certbot/ssl-dhparams.pem

```
cd certbot-etc
curl -s  https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf > options-ssl-nginx.conf

curl -s https://raw.githubusercontent.com/certbot/certbot/master/certbot/certbot/ssl-dhparams.pem > ssl-dhparams.pem
```



#### nginx.conf 개선

- http://로 접속시에는 https://로 redirect
- https://로 접속 지원을 위해, 443 포트 오픈 및 ssl 프로토콜 지원을 위한 

```
user nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" "$request_uri" "$uri"'
                      '"$http_user_agent" "$http_x_forwarded_for"';    
    access_log  /var/log/nginx/access.log  main;
    sendfile on;
    keepalive_timeout 65;

    upstream docker-web {
        server nginx:80;
    }

    server {
        listen 80;
        server_name familyzoa.com www.familyzoa.com;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /usr/share/nginx/html;
                try_files $uri =404;
        }

        location / {
                return 301 https://$host$request_uri;
        }    
    }


    server {
        listen 443 ssl;
        server_name familyzoa.com www.familyzoa.com;
        
        ssl_certificate /etc/letsencrypt/live/familyzoa.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/familyzoa.com/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf; # 보안 강화를 위한 옵션 추가
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;   # 보안 강화를 위한 옵션 추가

        location / {
            proxy_pass         http://docker-web;       # docker-web 컨테이너로 포워딩
            proxy_redirect     off;                     # 서버 응답 헤더의 주소 변경 (불필요)
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
	    proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }

}

```

## HTTPS 웹페이지 및 워드프레스 블로그 서비스 구현

- 다음과 같이 4개의 서버 컨테이너 필요
  - Nginx Proxy(HTTPS)
  - 내부 nginx 서버(나의 웹페이지)
  - 내부 wordpress 서버 (나의 블로그)
  - 내부 mysql 데이터베이스 (워드프레스용)

- 여기에 certbot을 수행하는 컨테이너 필요 

### 설정 개선

- nginx/nginx/conf 설정 개선

```
user nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" "$request_uri" "$uri"'
                      '"$http_user_agent" "$http_x_forwarded_for"';    
    access_log  /var/log/nginx/access.log  main;
    sendfile on;
    keepalive_timeout 65;


    upstream docker-wordpress {
        server wordpress:80;
    }

    upstream docker-web {
        server nginx:80;
    }

    server {
	listen 80;
	listen [::]:80;
	server_name fun-coding.xyz www.fun-coding.xyz;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /usr/share/nginx/html;
        }

        location / {
        	return 301 https://$host$request_uri;
	}
    }

    server {
	listen 443 ssl;
	server_name fun-coding.xyz www.fun-coding.xyz;

        ssl_certificate /etc/letsencrypt/live/fun-coding.xyz/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/fun-coding.xyz/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
	ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        location /blog/ {
            #rewrite ^/blog(.*)$ $1 break;
            proxy_pass         http://docker-wordpress;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }

        location / {
            proxy_pass         http://docker-web;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_set_header   X-Forwarded-Proto $scheme;
        }
    }
}

```



## 인증서 갱신

- certbot은 인증서 확인 후, 컨테이너가 중지되므로, 해당 컨테이너를 주기적으로 실행해주면 됨
- --dry-run 옵션으로 충분히 테스트 후, 다음과 같이 --force-revewal을 넣어줌 

```
  certbot:
    depends_on:
      - nginxproxy 
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./certbot-etc:/etc/letsencrypt
      - ./myweb:/usr/share/nginx/html
    command: certonly --webroot --webroot-path=/usr/share/nginx/html --email test@test.com --agree-tos --no-eff-email --keep-until-expiring -d familyzoa.com -d www.familyzoa.com --force-renewal 

```

### crontab 설정

- 리눅스나 맥에서 주기적으로 명령을 실행하는 것 

```
crontab -e
```

```
PATH=/usr/local/bin

* * * * * docker-compose -f /home/ubuntu/09_HTTPS_NGINX/docker-compose.yml restart certbot >> /home/ubuntu/09_HTTPS_NGINX?cron.log 2>&1
```

#### Crontab 주요 명령

- crontab -e
  - 크론탭 설정 입력 파일(텍스트 에디터 사용 태스크 추가/수정/삭제)
- crontab -l
  - 현재 크론탭에 설정되어 있는 내용 확인

#### Crontab 설명

- \* 5개로 이뤄져 있음

  ```(
  *         *        *        *        *
  분(0~59) 시간(0~23) 일(1~31) 월(1~12) 요일(0~7)
  ```

  - 요일에서 0과 7은 일요일 1~6은 월요일에서 토요일 

- 특정시간에 실행 -매주 월요일 오전 6시 40분에 실행

  ```
  40 6 * * 1 /root/scripts/status_check.sh
  ```

- 반복 실행 - 매일 매시간 0분, 20분, 40분에 실행

  ```
  0,20,40 * * * * /root/scripts/status_check.sh
  ```

- 범위 실행 - 매일 오전 6시 10분부터 40분까지 매분 실행

  ```
  10-40 6 * * * /root/scripts/status_check.sh
  ```

- 간격 실행 - 매 20분마다 실행

  ```
  */20 * * * * /root/scripts/status_check.sh
  ```

- 특정 여러 시각 실행 - 10일에서 12일까지 4시,5시,6시 매 20분마다 실행

  ```
  */20 4,5,6 10-20 * * /root/scripts/status_check.sh
  ```

#### crontab 실행 팁

- 로그 남겨두기
- 단, 로그가 많이 쌓이면, 저장공간이 꽉 차게 되고, 컴퓨터 다운 또는 비정상동작을 보일 수 있으므로, 주기적으로 삭제 

```
*/20 * * * * /root/scripts/status_check.sh >> /var/log/status_check.log 2>&1
* * 1 * * rm -rf /var/log/status_check.log 2>&1
```

