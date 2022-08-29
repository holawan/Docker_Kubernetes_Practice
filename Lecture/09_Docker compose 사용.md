# Docker compose

- Docker Compose는 여러 컨테이너를 모아서 관리하기 위한 툴
- 웹서비스는 프론트엔드 서버, 데이터베이스 서버, 백엔드 서버로 이루어져 있는 경우가 많음
    - 각각을 docker 컨테이너로 작성하고, 연결하여 동작하기 때문에, Docker Compose와 같은 컨테이너 관리 툴이 필요함
- 더 나아가 서비스 규모가 커지면, 복수의 컨테이너를 유지하고 관리해야 하며, 이를 위해 쿠버네티스 등의 관리 툴이 사용됨
    - docker와 docker compose를 잘 다룰 수 있으면, 기본적인 서비스 구현이 가능하며,
    - docker와 docker compose에 대한 탄탄한 이해가 바탕이 되어야, 추후 필요시 쿠버네티스도 원활하게 익히고 활용할 수 있음

## Docker Compose 작성 기본

- Docker Compose는 docker-compose.yml 파일을 작성하여 실행할 수 있음
- docker-compose.yml 파일은 YAML(야멜) 형식으로 작성함 



#### 참고 YAML 문법

- IT에서는 데이터를 구조화하는 다양한 문법이 있음
- 대표적인 데이터 구조화 문법은 JSON, XML, CSV등이 있음

- 이외에 일부 YAML 문법을 사용하는 케이스가 있음
    - key와 value, 그리고 들여쓰기를 중심으로 하는 문법. 
    - 중급 이상 기술에서 YAML을 사용하는 경우가 있음

##### YAML 기본 문법

- \#: 해당 라인을 주석처리

- --- : 문서의 시작을 나타냄 (옵션)

- ... : 문서의 끝을 나타냄 (옵션)

- `key :value` : key에 대한 값(value)

- 자료형

    - int,string,boolean 지원
        - int_type : 1
        - string_type : "문자열"
        - boolean_type : true or false 

- 데이터 표현

    - JSON 포맷과 비교하여 쉽게 이해 가능 

    - JSON

        ```json
        # JSON 포맷
        {
            "holawan" : [
            	"name" : "wan",
            	"job" : [
            		"student",
            		"software developer"
            		]
            ],
        	"hello" : {
            "company" : false,
            "tech" : {
            	"web-front" : ["vue"],
        		"backend" : ["python","C"]
        		}
        	}
        }
        ```

        - 리스트는 들여쓰기 (보통 스페이스 2칸 or 4칸)으로 표시

    - YAML

        ```yaml
        ---
        #문서 시작 
        	holawan:
        		- name : wan
        		- job :
        			- student
        			- software developer
            hello :
            	company :false
            	tech :
            		web-front :
            			- vue
            		backend :
            			-python
            			-C
        ...
        ```

        - '-' (하이픈)이 있으면 리스트 형태, 아니라면 객체 형태 

    - 참고 : 줄바꿈 

        - 줄바꿈 표시 : "|"는 마지막 줄바꿈을 포함

        ```json
        {
        	"newline" : "1라인\n\n2라인\n\n\3라인\n"
        }
        ```

        ```yaml
        newline: |
        		1라인 
        		
        		2라인
        		
        		3라인 
        ```

        - 줄바꿈 표시 "|-"는 마지막 줄바꿈 제외 

        - 줄바꿈 표시 ">"는 중간에 있는 줄바꿈을 아예 무시함(마지막 줄바꿈은 포함)

            - 따라서 다음과 같이 YAML로 작성하면, 그 다음과 같이 JSON으로 작성한 것과 동일하게 됨

            ```yaml
            newline : >
            	1라인
            	
            	2라인 
            	
            	3라인 
            ```

            ```json
            {
                "newline" : "1라인\n2라인\n3라인\n"
            }
            ```

            

## docker-compose.yml로 이해하는 Docker compose 사용법 1 

- Docker Compose 명령은 기본적으로 Dockerfile의 명령에 기반 

```yaml
version : '3'

services:
  db:
    image: mysql
    restart: always
    volumes: 
      - ./mysqldata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=dbname
    ports:
      - "3307:3306"
```

- 기본적으로 다음과 같은 4가지의 큰 카테고리로 작성하며, 이 중에서 보통 version과 services만 설정하여 많이 사용함

    - volumes는 각 컨테이너 설정에서의 volumes로 선언할 수 있고, networks는 컨테이너 간 네트워크 분리를 위한 추가 설정 부분임

    ```yaml
    # Docker Compose 파일 포맷 버전 지정
    version: '3'
    
    # 컨테이너 설정 
    services:
    
    #컨테이너에서 사용하는 volume 설정으로 대체 가능(옵션)
    volumes:
    
    #컨테이너간 네트워크 분리를 위한 추가 설정 부분(옵션)
    networks:
    
    ```

    - 보통 하나의 도커에 여러개의 도커 컨테이너를 넣을 때 networks를 자주 사용하지는 않음 

### version

- Docker Compose 파일 포맷 버전 지정
- docker 버전에 따라 지원하는 Docker Compose 버전이 있으며, 기본적으로는 버전 3으로 사용하는 것이 일반적임
    - 예를 들어, 3.8과 같이 최신 버전을 사용할 경우, 최신 docker 버전에서만 지원이 됨
    - 3.8과 같은 최신 버전에서만 지원하는 Docker Compose 특수 문법까지 사용할 일은 많지 않기 때문
    - docker를 yaml로 변경해주는 사이트 
        - https://www.composerize.com/
    - 버전별 호환성 사이트
        - https://docs.docker.com/compose/compose-file/compose-versioning/

```
version: "3"
```



### services

- 위 항목 아래에서 여러개 또는 하나의 컨테이너를 설정함
- 컨테이너를 정의하는 명령 

#### Image 

- 다음 코드에서 db는 컨테이너 이름을 정의하는 것
- db라는 이름의 컨테이너 작성 시, DockerHub에 있는 이미지를 사용할 경우, image를 설정하면 됨
    - mysql이라는 Docker Hub에 있는 이미지를 사용하겠다는 의미 

```
services:
  db:
    image: mysql
```

#### restart

- 컨테이너가 다운되었을 경우, 항상 재시작하라는 설정
- 서버는 24시간 동작해야 하므로, 언제든 다운될 수 있으며, 이를 위한 모니터링/유지보수 작업이 필요함
- 위 옵션을 통해 왠만한 케이스에서는 계속 동작할 수 있으며, 특정 서비스의 경우 아무런 유지보수 없이도 1년 이상 동작 가능 

```yaml
services:
  db:
    image: mysql
    restart: always
```



### volumes

- docker run 옵션 중 , volume(-v) 옵션과 동일한 역할
- 여러개의 volume을 지정할 수 있기 때문에, 리스트 처럼 작성

```
services:
  db:
    image: mysql
    restart: always
    volumes:
      - ./mysqldata:/var/lib/mysql
```



### environment

- Dockefile의 ENV 옵션과 동일한 역할

```yaml
services:
  db:
    image: mysql
    restart: always
    volumes: 
      - ./mysqldata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=dbname
```

- 다음 env_file 옵션으로 환경변수값이 들어가 있는 파일을 읽어들일 수도 있음
    - 패스워드 등 보안이 필요한 부분을 docker compose 보다는 별도의 파일로 작성하여, env_file 옵션으로 읽어들이는 방식을 쓰는 경우도 많음

```
services:
  db:
    image: mysql
    restart: always
    volumes: 
      - ./mysqldata:/var/lib/mysql
    env_file:
      - ./mysql.env
```

- env_file 포맷

    ```bash
    $ cat mysql.env
    MYSQL_ROOT_PASSWORD=password
    MYSQL_DATABASE=dbname
    ```



### ports

- docker run의 -p 옵션과 동일한 역할
- YAML 문법에서 aa:bb와 같이 작성하면, 시간으로 해석하기 때문에, 쌍따옴표를 붙여줘야 함

```yaml
version : '3'

services:
  db:
    image: mysql
    restart: always
    volumes: 
      - ./mysqldata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=dbname
    ports:
      - "3307:3306"
```



## Docker Compose 실행/중지하기

```
version : '3'

services:
  db:
    image: mysql
    restart: always
    volumes: 
      - ./mysqldata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=dbname
    ports:
      - "3307:3306"
```

### 실행 명령 (백그라운드에서)

- 보통 -d 옵션을 사용하며, -d 옵션은 백그라운드 실행을 의미함 

```
docker-compose up -d
```

- mysqldata 폴더도 생성됨

    ```
    ubuntu@ip-172-31-42-165:~/01_DockerCompose$ ls
    docker-compose.yml  mysqldata
    ```

    - mysql이 생성되면서 db관련 정보들이 복사되어 형성됨 

- 이미지 재빌드가 필요하면 --build 옵션을 추가해야함

    - 그렇지 않으면, 이미 작성된 이미지를 사용하게 됨

    ```
    docker-compose up --build -d 
    ```

### Docker Compose 중지 명령

```
docker-compose stop
```

- 컨테이너가 중지되지만 살아있음

    ```
    ubuntu@ip-172-31-42-165:~/01_DockerCompose$ docker ps -a
    CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS                     PORTS     NAMES
    a1931d17faf8   mysql:5.7   "docker-entrypoint.s…"   3 minutes ago   Exited (0) 5 seconds ago             01_dockercompose_db_1
    ```

    

#### Docker compose에서 사용하는 컨테이너 삭제 명령

- docker-compose up으로 생성된 컨테이너 삭제

```
docker-compose down
```

- docker ps -a확인

    ```
    ubuntu@ip-172-31-42-165:~/01_DockerCompose$ docker-compose down
    Removing 01_dockercompose_db_1 ... done
    Removing network 01_dockercompose_default
    ubuntu@ip-172-31-42-165:~/01_DockerCompose$ docker ps -a
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
    ```

- 테스트

    ```
    #Docker Compose실행
    $docker-compose up -d
    #실행 중 컨테이너 확인
    $docker ps 
    
    #컨테이너 삭제
    $ docker-compose down
    
    #컨테이너 확인(삭제되었으므로 아무것도 나오지 않음)
    $ docker ps 
    ```

    
