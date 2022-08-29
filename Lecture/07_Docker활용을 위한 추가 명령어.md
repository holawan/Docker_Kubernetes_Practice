# Docker 활용을 위한 추가 명령 익히기

## Docker 조사하기

### Docker history

- 이미지 히스토리 확인 

```dockerfile
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

```

- 이미지 생성

```
docker build --tag myweb_history -f Dockerfile_history ./
```

- 이미지 히스토리 확인

```
docker history myweb_history 
```

- 위와 같이 FROM, RUN, COPY, ENTRYPOINT 등 명령에 따라 layer가 생성됨을 확인할 수 있다.

### docker cp

- 컨테이너에 특정 파일을 호스트 PC로 가져오는(꺼내오는) 명령
- 특정 파일 확인을 위해 활용

- 컨테이너 작성

```
docker run -dit -p 9999:80 --name httpd_history --rm myweb_history
```

- apahce2 설정 파일 가져오기

```
docker cp httpd_history:/etc/apache2/sites-available/000-default.conf ./
```

- 반대로 호스트 PC에서 컨테이너에 특정 파일을 넣을 수도 있음

```
#컨테이너에 넣기 
docker cp ./000-default.conf httpd_history:/etc/apache2/sites-available/000-default.conf

#컨테이너 접속하기 
docker exec -it httpd_history /bin/bash

#컨테이너 안에서 다음 명령으로 변경된 파일이 들어갔음을 확인하기

cd /etc/apache2/sites-available/
cat 000-default.conf

#apache2 재실행 
apache2ctl restart 

exit
```

### docker commit 

- 컨테이너 변경사항을 이미지 파일로 생성
- 컨테이너에서 작업을 할 때, 그 작업은 컨테이너를 삭제할 때 완전 삭제된다. 
- 하지만 이 작업을 레이어로 만들어 새로운 이미지를 만들 수 있다. 

```
docker commit 옵션 컨테이너ID_또는_이름 이미지이름[:태그]
```

- 예 

    ```bash
    docker exec -it httpd_history /bin/bash
    root@ee23c120b003:/# cd /etc/apache2/sites-available/
    root@ee23c120b003:/etc/apache2/sites-available# vi 000-default.conf
    bash: vi: command not found
    ```

    - 컨테이너 안에서 파일을 변경하려 햇으나, vi 에디터가 없는 상황 
    - Vim 설치 

    ```
    apt-get update 
    apt-get install vim
    ```

    - vim 설치로 이제 컨테이너 내부의 수정을 할 수 있지만, 컨테이너가 삭제되면 vim은 없어진다.
    - 이 기능을 나중에도 유지하고 싶을 때 커밋을 이용할 수 있다.

    ```
    docker commit -m "add vim" httpd_history httpd_history_newvi
    ```

    ```
    IMAGE          CREATED          CREATED BY
                     SIZE      COMMENT
    058b63d106e9   6 seconds ago
                     49.6MB    add vim
    7530a3f9f505   14 minutes ago   /bin/sh -c #(nop)  ENTRYPOINT ["/usr/sbin/ap…   0B
    ```

    - add vim이라는 이미지가 생긴것을 알 수 있음 

    - 컨테이너를 삭제하고, 아까 생성한 이미지로 다시 컨테이너 만들어보기 

        ```
        docker run -d -p 9999:80 --name mywebserver --rm httpd_history_newvi
        ```

        ```
        root@ee23c120b003:/# cd /etc/apache2/sites-available/
        root@ee23c120b003:/etc/apache2/sites-available# vi 000-default.conf
        ```

    - vi가 설치되어 있으며 수정한 기록도 남아있음을 알 수 있다.

### docker diff

- 컨테이너가 실행되면서 본래의 이미지와 비교해서 변경된 파일 목록 출력

| 기호 | 설명                    |
| ---- | ----------------------- |
| A    | 파일 또는 디렉토리 추가 |
| D    | 파일 또는 디렉토리 삭제 |
| C    | 파일 또는 디렉토리 수정 |

```
ubuntu@ip-172-31-42-165:~$ docker diff mywebserver 
C /etc
C /etc/ld.so.conf
C /run
C /run/apache2
C /run/apache2/apache2.pid
C /root
C /root/.bash_history
A /root/.viminfo
C /var
C /var/log
C /var/log/apache2
C /var/log/apache2/error.log
C /var/log/apache2/access.log
```

- txt 파일 만들어보기

```
ubuntu@ip-172-31-42-165:~$ docker exec -it mywebserver /bin/bash
root@7e4325b3ac3b:/# cd ~
root@7e4325b3ac3b:~# ls
root@7e4325b3ac3b:~# vi test.txt
root@7e4325b3ac3b:~# exit
exit

ubuntu@ip-172-31-42-165:~$ docker diff mywebserver
C /etc
C /etc/ld.so.conf
C /var
C /var/log
C /var/log/apache2
C /var/log/apache2/access.log
C /var/log/apache2/error.log
C /root
C /root/.bash_history
A /root/.viminfo
A /root/test.txt
C /run
C /run/apache2
C /run/apache2/apache2.pid
```

### docker inspect 

- 이미지와 컨테이너 세부 정보 확인

### docker logs 

- 컨테이너의 출력결과(STDOUT)를 확인
