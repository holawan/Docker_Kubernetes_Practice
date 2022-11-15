# Docker file

## Docker file이란?

- docker 이미지를 작성할 수 있는 기능
- Dockerfile 문법으로 이미지 생성을 위한 스크립트를 작성할 수 잇고, 이를 기반으로 이미지 생성
- 나만의 이미지를 생성할 수 있고, 배포를 위해서도 사용 가능

## 1. Dockerfile 문법

- Dockerfile은 텍스트 파일 형식이므로, 각자사용하는 어떤 에디터로든 작성할 수 있음

- Dockerfile 기본 문법

    - 기본적으로는 간단히 명령과 인자로 이루어짐
    - 명령은 통상적으로 대문자로 작성 (소문자로 작성해도 문제없지만, 명령임을 구별하기 위해 일반적으로 대문자로 작성)

    ```dockerfile
    명령 인자 
    ```
    

### 주요 명령

| 명령       | 설명                                                         |
| ---------- | ------------------------------------------------------------ |
| FROM       | 베이스 이미지 지정 명령 (예: FROM httpd:alpine)              |
| LABEL      | 버전 정보, 작성자와 같은 이미지 설명을 작성하기 위한 명령 (예 : LABEL version="1.0.0") |
| CMD        | docker 컨테이너가 시작할 때, 실행하는 쉘 명령을 지정하는 명령. RUN과 비슷하지만, RUN은 이미지 작성 시 실행하는 명령이고, CMD는 컨테이너를 시작할 때 실행하는 명령임 CMD ['python','app.py] |
| RUN        | 쉘 명령을 실행하는 명령 (예 : RUN["apt-get","install","nginx"]). RUN은 이미지 작성시 실행되며, 일종의 새로운 이미지 layer를 만드는 역할을 함 |
| ENTRYPOINT | docker 컨테이너가 시작할 때, 실행하는 쉘 명령을 지정하는 명령. docker run 커멘드 실행 시, 별도 명령어도 넣을 수 있는데, 이 때 CMD 명령은 해당 명령으로 덮어쓰워짐. ENTRYPOINT로 지정한 명령은 docker run 커멘드 실행시 함께 넣어진 별도 명령어가 있더라도, 덮어씌워지지 않고 실행됨 |
| EXPOSE     | docekr 컨테이너 외부에 오픈할 포트 설정 (예: EXPORT 8080)    |
| ENV        | docker 컨테이너 내부에서 사용할 환경변수 지정 (예 : ENV PATH /usr/bin:$PATH) |
| WORKDIR    | docker 컨테이너에서의 작업 디렉토리 설정                     |
| COPY       | 파일 또는 디렉토리를 docker 컨테이너에 복사. ADD와 달리 URL은 지정할 수 없으며, 압축파일을 자동으로 풀어주지 않음 (예: COPY test.sh /root/test.sh) |

### 추가 명령 (알아두면 좋음)

| 명령    | 설명                                                         |
| ------- | ------------------------------------------------------------ |
| ADD     | ADD와 COPY 명령은 유사하지만, COPY 명령이 보다 명시적이므로, COPY 명령을 사용<br />파일, 디렉토리, 또는 특정 URL의 데이터를 DOCKER 이미지에 추가 (예 ADD file /var/www/html).<br />추가할 파일이 TAR 압축 파일일 경우, 자동으로 압축을 풀어줌 (이 기능이 압축파일을 그대로 넣고 싶을 때 문제가 됨)<br />동일한 이름의 파일 또는 디렉토리가 이미 docker 이미지에 있을 시에는 덮어쓰우지 않음 (예: ADD test.sh /root/test.sh) |
| SHELL   | 쉘 프로그램 지정 명령어이지만, CMD 등으로 대체 가능하므로 참고로만 알아둠 (예: SHELL ['/bin/bash','-c']) |
| ARG     | dockerfile 내에서 필요한 변수 설정. docker 이미지/컨테이너에서 사용하는 환경 변수를 설정하는 ENV와 달리, dockerfile 스크립트 작성을 위해 필요한 변수를 설정 (예: ARG env=dev) |
| USER    | docker 이미지 및 컨테이너에서 작업을 하는 사용자 ID를 지정함 (예 : USER dave) |
| ONBUILD | 생성한 이미지를 기반으로, 새로운 이미지를 생성 시 실행하는 명령을 지정(예: ONBUILD ADD myweb.tar /var/www/html) |

- dockerfile에서도 주석을 사용할 수 있음

    ```dockerfile
    # Dockerfile 석은 #을 쓰면 , 해당 라인은 주석
    ```

### FROM

- 베이스 이미지 지정 명령

- 반드시 Dockerfile에 작성해야 하는 명령

    ```dockerfile
    #Dockerfile 파일명으로 다음과 같이 작성
    FROM alpine
    ```

#### Dockerfile 이미지 작성

```
docker build 옵션 Dockerfile_경로
```

##### 주요 옵션

| 옵션          | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| -t 또는 --tag | 이미지 이름 설정. 이미지 이름은 저장소(DockerHub ID)/이미지이름:태그 와 같이 작성할 수 있음<br />(저장소 이름 및 태그 이름은 작성안해도 되며, 태그 이름이 없을경우 default로 latest가 태그로 붙여짐) |
| -f            | 이미지 빌드시 디폴트로 Dockerfile 파일명으로 된 파일을 찾아서, 이미지를 빌드함. 그 외의 파일명으로 이미지를 빌드할 경우 해당 옵션을 사용해서 파일명을 지정할 수 있음 |
| --pull        | FROM 으로 지정된 이미지는 한 번 다운로드 받으면, 이미지 생성시마다, 새로 다운로드 받지 않고, 다운로드 받은 이미지를 사용함.<br />해당 옵션은 이미지 생성시마다 새로 다운로드를 받는 옵션임. --pull=true와 같이 작성하며, 사용함. Dockerhub에 베이스 이미지를 수시로 업데이트하고 ,이를 기반으로 새로운 이미지 생성 시 자주 사용할 수 있는 옵션 |

- 테스트

    - 위에서 작성한 Dockerfile이 있는 동일 경로에서 다음과 같이 명령
    - --tag test는 이미지 이름을 test로 설정한 것이므로, 디폴트 태그가 붙음
    - **마지막은 "."은 현재 폴더를 나타내는 것이고, 즉 현재 폴더의 Dockerfile을 실행하라는 뜻**
        - 보다 명시적인 표기를 위해 "." 대신 "./"와 같이 쓰는 것을 추천 

    - dockerfile

        ```dockerfile
        FROM alpine
        ```

    - image 생성 

        ```bash
        docker build --tag myimage:1 .
        docker build --tag myimage:1.1 ./  
        ```

- -f 옵션 테스트

    ```bash
    cp Dockerfile test_dockerfile
    docker build --tag test2 -f test_dockerfile ./
    docker images
    ```

- --pull 옵션 테스트

    - pull 옵션이 있으면 이미지가 있는지에 관련 없이 새로 명시한 이미지를 다운받음 

    ```bash
    docker build --tag test2 -f test_dockerfile ./ --pull=true
    ```

    

### LABEL

- LABEL 은 \<key>=\<value> 형식으로 메타 데이터를 넣을 수 있는 기능
- 보통 저자, 버전, 설명, 작성일자 등을 각각 key 이름을 정하고, 값을 넣는 경우가 있음
- git에서 pull 받고 Dockerfile 실행할 때 이름이 같으면 갱신이 안되는 경우가 있었는데, 해당 경우에서 LABEL을 수정하면서 해도 될듯 
    - 예전에는 그냥 RUN echo "be:1.0" 이런식으로 했는데.. 

```dockerfile
#Dockerfile 파일명으로 다음과 같이 작성
FROM alpine
LABEL maintainer="asdf134652@gmail.com"
LABEL version="1.0.0"
LABEL description="A test Docekr image to understanding"
```

- 테스트

    ```
    docker build --tag myweb .
    ```



### COPY

- Dockerfile 생성

    - 하부 폴더 2021_DEV 폴더에 이미지에 추가할 파일들을 넣어놓은 상태에서 다음과 같이 Dockerfile 생성
        - FROM에 베이스 이미지를 httpd:alpine으로 수정 
    - Volume 명령도 있지만, 호스트 PC의 특정 폴더를 컨테이너 내부 폴더로 연결하는 옵션은 -v 옵션으로만 가능하며, VOLUME 명령은 컨테이너 내부의 특정 폴더를 위한 볼륨을 생성하기 위해서만 사용됨
        - 예1: VOLUME /mydata
        - 예2: VOLUME ["/mydata1", "/mydata2"]
    - docker file 하부의 2021_DEV_HTML을 /usr/local/apache2/htdocs에 복사해라 ! 

    ```
    FROM httpd:alpine
    LABEL maintainer="asdf134652@gmail.com"
    LABEL version="1.0.0"
    LABEL description="A test Docekr image to understanding"
    
    COPY ./2021_DEV_HTML /usr/local/apache2/htdocs
    ```

- 테스트

    ```
    docker build --tag myweb .
    ```

    

    - 다음과 같이 실행 후 port에 접속해보기 

    ```
    docker run -d -p 9999:80 --name apacheweb myweb
    ```

- docker 이미지 조사하기

    - 다음과 같이 httpd:alpine 이미지와 직접 Dockerfile로 작성한 이미지를 조사해보기

    ```
    docker inspect myweb
    ```

### CMD

- 다음 세가지 형태로 CMD 명령을 작성할 수 잇음

    - 명령어, 인자를 리스트처럼 작성하는 형태(해당 방식을 docker에서는 추천함)

        - 다음 예에서 echo 명령만 써줄 경우, 쉘에서 실행하지 않고, 직접 해당 명령이 실행되므로, 쉘에서 실행할 때, 적용되는 쉘의 환경변수 등 적용되지 않음(따라서, 가장 정확하게는 명확히 쉘까지 지정해서 명령을 실행해주는 것이 좋음)
        - /bin/sh -c 명령은 /bin 디렉토리에 있는 sh 프로그램 (기본 쉘 프로그램)을 실행하되,
            - -c 옵션은 쉘 명령을 터미널상에서 받지 않고, 인자로 받겠다는 의미임 
        - 쌍따옴표로 써야함 (홑따옴표를 쓰면 안됨)

        ```
        CMD ["executable", "param1", "param2", ...]
        #예
        CMD ["/bin/sh", "-c", "echo", "Hello"]
        ```

    - ENTRYPOINT 명령어에 인자를 리스트처럼 작성하여 넘겨주는 형태 

        - 엔트리포인트에 명령을 설정하면, CMD에는 해당 명령을 실행할 때 인자를 넣어주면, 엔트리포인트 뒤의 인자로 추가됨. 

        ```
        CMD ["param1", "param2", ...]
        ```

    - 쉘 명령처럼 작성하는 형태

        ```
        CMD <command> <param1> <param2>
        ```

- CMD는 하나의 Dockerfile에서 한 가지만 설정되며, 만약 CMD 설정이 여러개일 경우, 맨 마지막에 설정된 CMD 설정만 적용됨 

- httpd:alpine 기반 Dockerfile 작성하기

    ```dockerfile
    # test_dockerfile 파일명으로 다음과 같이 작성
    FROM httpd:alpine
    LABEL maintainer="asdf134652@gmail.com"
    LABEL version="1.0.0"
    LABEL description="A test Docekr image to understanding"
    
    COPY ./2021_DEV_HTML /usr/local/apache2/htdocs
    
    CMD ["/bin/sh", "-c", "httpd-foreground"]
    ```

- 테스트

    - 지난 컨테이너 포트나 이름 겹치면 지우기 

    ```
    docker build --tag test2 -f test_dockerfile
    ```

    ```
    docker run -d -p 9999:80 --name httpdweb2 test2
    docker ps 
    ```

#### 가끔 사용하는 Docker 명령 1 

- 컨테이너 에러 또는 출력 결과 확인하기
- 컨테이너의 표준 출력이나 표준 에러를 확인하는 것이다.

```bash
docker logs 컨테이너ID_또는_이름
#예
docker logs httpweb1
```

#### 가끔 사용하는 docker 명령 2 : 컨테이너 즉시 중지하기

- docker stop은 즉시 컨테이너를 중단하지 않고, 현재 시행중인 단계까지는 기다린 후에 중지 함
- 이에 반해 docker kill은 즉시 컨테이너 중지

```
docker kill 컨테이너ID_또는_이름
```

#### CMD 변경해보기

```dockerfile
# test_dockerfile 파일명으로 다음과 같이 작성
FROM httpd:alpine
LABEL maintainer="asdf134652@gmail.com"
LABEL version="1.0.0"
LABEL description="A test Docekr image to understanding"

COPY ./2021_DEV_HTML /usr/local/apache2/htdocs

CMD ["/bin/sh"]
```

```
#이미지 생성
docker build --tag myweb2 -f test_dockerfile ./
#-dit 옵션으로 터미널 붙이고, 포트 오픈하고, 백그라운드로 실행
docker run -dit -p 9999:80 --name httpdweb2 --rm myweb2
```

- 위와 같이 변경할 경우, 접속 실패 
    - 우리가 만든 httpd:alpine 을 base로 한 dockerfile은 `httpd-foreground`라는 CMD 명령이 지정되어 있으나, 이가 실행되지 않음 

- 이미지 조사하기
    - CMD가 변경된 것을 확인할 수 있음
    - 터미널에 접속할 수 있었던 것은, -dit 옵션으로 터미널로 연결된 채로, 컨테이너 실행시 /bin/sh 프로그램이 실행되면서, 대기 상태로 되어있었기 때문 

- CMD 명령 덮어씌우기

    ```
    docker run -dit -p 9999:80 --name httpdweb2 --rm myweb2 /bin/sh -c httpd-foreground
    ```

- 컨테이너와 이미지 조사하기

    - Config.cmd 확인하면, 명령이 덮어씌워졌음을 알 수 있음

    ```
    docker inspect myweb2
    docker inspect httpdweb2
    ```

 

### ENTRYPOINT

- ENTRYPOINT는 docker run 시에 함께 넣어지는 CMD 명령에 덮어씌워지지 않고, 반드시 실행해야 하는 명령을 기입할 때 사용
    - 이 때, docker run 시 함께 넣어지는 명령은 ENTRYPOINT에 작성된 명령의 인자로 넣어지게 됨
    - 따라서, ENTRYPOINT에 컨테이너 실행 시, 반드시 실행되야 하는 명령을 넣고, 별도로 각 컨테이너 생성 시 필요한 인자는 docker run에 넣는 식으로 활용하기도 함 
    - 이미지에 명령을 박아넣는다.! 
    - 이미지에 ENTRYPOINT가 있다면 CMD의 인자 역할을 한다. 
- ENTRYPOINT로 변경해보기 

```
#Dockerfile_HTTPD3 파일명으로 다음과 같이 작성
FROM httpd:alpine
LABEL maintainer="asdf134652@gmail.com"
LABEL version="1.0.0"
LABEL description="A test Docekr image to understanding"

COPY ./2021_DEV_HTML /usr/local/apache2/htdocs

ENTRYPOINT ["/bin/sh"]
```

- 컨테이너와 이미지 조사하기
    - IP로 접근 불가
    - CMD는 null이고, Entrypoint에만 항목이 들어감

```
docker build --tag myweb3 -f Dockerfile_HTTPD3 ./

docekr inspect myweb3
```

- 다양한 경우 테스트해보기 

    - Dockerfile

        ```
        #Doickerfile_HTTPD3 파일명으로 다음과 같이 작성
        FROM httpd:alpine
        LABEL maintainer="asdf134652@gmail.com"
        LABEL version="1.0.0"
        LABEL description="A test Docekr image to understanding"       
        
        COPY ./2021_DEV_HTML /usr/local/apache2/htdocs
        
        ENTRYPOINT ["/bin/echo","hello"]
        ```

    - build

        ```
        docker build --tag myweb3 -f Dockerfile_HTTPD3 ./
        ```

    - 컨테이너 실행 

        ```
        docker run -dit -p 9999:80 --name myweb3 myweb3 /bin/sh hi
        ```

    - 로그 확인

        ```
        docker logs myweb3 
        hello /bin/sh hi
        ```

    - 결과 해석
        - entrypoint에 echo가 있기 때문에, hello라는 명령이 먼저 출력됨
        - docker run에 cmd 명령을 진행시켰기 때문에, entry point 인자로 인식되어 /bin/sh hi 가 출력됨 
        - **즉 엔트리포인트만 있는 dockerfile 이미지에 cmd를 인자로 주면 entrypoint인자로 들어간다.** 
        - CMD의 /bin/sh hi 두 명령은 Entrypoint의 인자로 추가되어 해당 형태
            - /bin/echo hello /bin/sh hi

### RUN

- docker는 이미지 생성시 , 각 단계를 layer로 나누어 작성함
    - 이를 통해 특정 단계 변경시, 전체 이미지를 다시 다운로드 받지 않아도 됨
- RUN 명령은 이미지 생성시, 일종의 layer를 만들 수 있는 명령으로, 보통 베이스 이미지에 패키지(프로그램)을 설치하여, 새로운 이미지를 만들 때 많이 사용
- 즉, base 이미지가 있을 때 추가적인 이미지를 올리려면 RUN 명령어를 사용

```
docker pull httpd:alpine
```

- 예: ubuntu 18.04 버전에 apache2 설치하고, 나만의 웹페이지 복사 후 웹서버 구동 

- 테스트

    - 기본 ubuntu 이미지에 apache2 올리기 

    ```
    #Doickerfile_HTTPD3 파일명으로 다음과 같이 작성
    FROM ubuntu:18.04
    LABEL maintainer="asdf134652@gmail.com"
    LABEL version="1.0.0"
    LABEL description="A test Docekr image to understanding"       
    
    RUN apt-get update #패키지(프로그램) 정보 업데이트
    RUN apt-get install -y apache2 #apache2 패키지(프로그램) 설치, 중간에 Y/N 묻는 단계가 나오면 모두 Yes로 하고 설치
    
    #apache2 디폴트 웹서버 설정은 /var/www/html/ 폴더의 웹페이지를 보여줌
    COPY ./2021_DEV_HTML /var/www/html/
    
    #apache2 웹서버 구동 명령은 다음과 같음 
    ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
    
    
    ```

    - 이미지 생성

        ```
        docker build --tag myweb4 -f Dockerfile_HTTPD3 ./
        
        docker run -dit -p 9999:80 --name httpdweb4 --rm myweb4
        ```

    - 이미지에 변경사항 주기 

        ```
        #Doickerfile_HTTPD3 파일명으로 다음과 같이 작성
        FROM ubuntu:18.04
        LABEL maintainer="asdf134652@gmail.com"
        LABEL version="1.0.0"
        LABEL description="A test Docekr image to understanding"       
        
        RUN apt-get update #패키지(프로그램) 정보 업데이트
        RUN apt-get install -y apache2 apt-utils #apache2 패키지(프로그램) 설치, 중간에 Y/N 묻는 단계가 나오면 모두 Yes로 하고 설치
        
        #apache2 디폴트 웹서버 설정은 /var/www/html/ 폴더의 웹페이지를 보여줌
        COPY ./2021_DEV_HTML /var/www/html/
        
        #apache2 웹서버 구동 명령은 다음과 같음 
        ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
        
        
        ```

### EXPOSE

- docker container의 특정 포트를 외부에 오픈하는 설정
    - docker run -p 옵션으로 설저할수도 있음
        - docker run -p 옵션은 컨테이너의 특정 포트를 외부에 오픈하고, 해당 포트를 호스트 PC 특정 포트와 매핑시킴
    - 이에 반해, EXPOSE는 컨테이너 생성시, 특정 포트를 외부에 오픈하는 것만 설정하는 것임
        - 따라서, 독립적으로 실행시키는 EXPOSE 옵션을 넣는다고 해서, 호스트 PC에서 해당 컨테이너에 포트 번호로 접속할 수 있는 것은 아님 

- 테스트

    ```
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
    ```

    ```
    docker build --tag myweb4 -f Dockerfile_HTTPD3 ./
    
    ```

    ```
    # -P 옵션을 쓰면,EXPOSE로 오픈된 포트에 호스트 PC의 랜더포트가 매핑됨
    docker run -P -d myweb
    ```



### ENV

- 컨테이너 내의 환경변수 설정
- 설정한 환경변수는 RUN, CMD, ENTRYPOINT 명령에도 적용됨
- MYSQL Dockerfile 작성 

```
# Dockerfile_MYSQL

FROM mysql:5.7

#mysql 슈퍼관리자인 root ID에 대한 password란에 원하는 패스워드 설정
ENV MYSQL_ROOT_PASSWORD=password
#dbname란에 원하는 데이터베이스 이름 설정
ENV MYSQL_DATABASE=dbname

#필요시 다음 설정도 가능 
ENV MYSQL_USER=user #user 란에 mysql 추가 사용자 ID 설정 
ENV MYSQL_PASSWORD=pw # pw란에 mysql 추가 사용자 패스워드 설정 
```

- 이미지 빌드 후 실행 

```
#docker 이미지 작성하기
docker build --tag mysqldb -f Dockerfile_MYSQL ./

docker run -d --name mydb mysqldb
```

- 컨테이너 접근

    ```
    docker exec -it mydb /bin/bash
    
    bash-4.2# mysql -u root -p 
    
    #패스워드 입력
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | dbname             |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    5 rows in set (0.00 sec)
    
    ```

    

### WORDIR

- RUN, CMD, ENTRYPOINT 명령이 실행될 디렉토리 설정

    ```
    FROM httpd:alpine
    
    WORKDIR /usr/local/apache2/htdocs
    
    CMD /bin/cat index.html
    ```

    ```
    docker build --tag httpd6 -f Dockerfile_HTTPD3 ./
    
    docker run -d --name httpd6 httpd6 
    
    docker logs httpd6 
    ```

    
