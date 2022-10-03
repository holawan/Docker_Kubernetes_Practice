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



### Docker Compose 실행/중지하기

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




## docker-compose.yml로 이해하는 Docker Compose 사용법 2 

- 기존에 작성한 docker-compose.yml에 컨테이너를 추가하며, 추가 문법 이해하기

```yaml
version: "3"

services:
  app:
    build:
      context: ./01_FLASK_DOCKER
      dockerfile: Dockerfile
    links:
      - "db:mysqldb"
    ports:
      - "80:8080"
    container_name: appcontainer
    depends_on:
    - db
    
  db:
    image: mysql:5.7
    restart: always

    volumes:
      - ./mysqldata:/var/lib/mysql 
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=dbname
    ports:
      - "3306:3306"
    container_name: dbcontainer
```

### build

- 이미지를 Dockerfile 기반으로 작성시 사용
    - context: Dockerfile이 있는 디렉토리
    - dockerfile: Dockerfile명 

- Dockerfile을 별도로 만들고, 그 파일로 이미지를 만들게 하는 옵션
- context는 어느 폴더인지 명시해주는 것 



### links

- 컨테이너 내부에서, 다른 컨테이너를 접속하고 싶을 때 사용
- 다음 YAML 코드에서, db 컨테이너를 app 컨테이너에서 사용하고 싶을 때 ,

```
version: "3"

services:
  app:
    build:
      context: ./01_FLASK_DOCKER
      dockerfile: Dockerfile
    links:
      - "db:mysqldb"
    ports:
      - "80:8080"
    container_name: appcontainer
    depends_on:
    - db
    
  db:
    image: mysql:5.7
```

- 다음과 같이 작성하면, db라는 이름으로 컨테이너 접속 가능

```
version: "3"

services:
  app:
    build:
      context: ./01_FLASK_DOCKER
      dockerfile: Dockerfile
    links:
      - "db:mysqldb"
    ports:
      - "80:8080"
    container_name: appcontainer
    depends_on:
    - db
    
  db:
    image: mysql:5.7
```

- 다음과 같이 작성하면, mysqldb 또는 db 이름으로(둘 다 가능) 컨테이너 접속 가능
    - 역시 aa:bb와 같은 형태는 YAML 문법에서 날짜로 인지하기 때문에, 쌍따옴표로 작성해야 함 

```
version: "3"

services:
  app:
    build:
      context: ./01_FLASK_DOCKER
      dockerfile: Dockerfile
    links:
      - "db:mysqldb"
    ports:
      - "80:8080"
    container_name: appcontainer
    depends_on:
    - db
    
  db:
    image: mysql:5.7
```

### depends_on

- app 컨테이너에서는 mysqldb가 있어야 db를 사용할 수 있고, 서버가 정상적으로 실행될 수 있다. 하지만, 어느 컨테이너가 먼저 빌드될지 모르는 상황.
- depends_on 옵션을 주면, 해당 리스트에 있는 컨테이너부터 생성 후 다음 컨테이너를 생성한다. 

- 여러 컨테이너를 Docker Compose로 실행할 경우, 각 컨테이너가 실행을 시작하는 시점이 미묘하게 다를 수 있다.
- 따라서, 특정 컨테이너가 시작하자마자, 바로 다른 컨테이너를 접속하도록 코드를 작성하면, 시점에 따라 접속 불가 에러가 발생할 수 있다.
- 이를 위한 옵션으로 depends_on 옵션이 있지만, 해당 옵션도 컨테이너 실행 순서만 제어하고, 컨테이너가 ready 상태가 될 때까지를 명확히 제어하는 것은 아니므로 depends_on 옵션이 기대한대로 동작하지 않을 수 있다.
- 예를들어, 위와 같이 작성하면, db 컨테이너가 app 컨테이너보다 먼저 실행이 되지만, ready 사이태는 어느 컨테이너가 먼저 될지 알 수 없음 
    - 각 컨테이너 설정마다, 실제 프로세스가 실행되는 시점은 다를 수 있기 때문

### Flask Mysql test

#### 테스트 환경 셋업

- Flask file

```python
from flask import Flask
import pymysql

app = Flask(__name__)

MYSQL_HOST = 'mysqldb'
MYSQL_PORT = 3306

def conn_mysqldb():
    MYSQL_CONN = pymysql.connect(
        host=MYSQL_HOST,
        port=MYSQL_PORT,
        user='root',
        passwd='password',
        db='dbname',
        charset='utf8'
    )
    return MYSQL_CONN


@app.route('/')
def hello_world():
    mysql_db = conn_mysqldb()
    db_cursor = mysql_db.cursor()
    sql = "SHOW TABLES"
    # print (sql)
    db_cursor.execute(sql)
    user = db_cursor.fetchone()
    print (user, MYSQL_HOST)
    return 'SUCCESS'


if __name__ == '__main__':
    app.run(host='0.0.0.0', port='8080')

```

- Flask Dockerfile
    - 다음과 같이 flask Dockerfile 작성
        - 아나콘다 도커로 continumio/miniconda 이미지 많이 사용
            - 아나콘다 풀패키지에 포함된 프로그램이 많기 때문에, 기본 패키지만설치
            - flask, pymysql을 위해 필요한 라이브러리 설치 

```dockerfile
#아나콘다 공식 이미지 
FROM continuumio/miniconda 
#현재폴더에 있는 코드를 내부에 /app이라는 폴더를 만들어 거기로 옮김 
COPY ./ /app

#/app으로 이동 
WORKDIR /app


RUN pip install flask pymysql cryptography
#flask 서버 실행 
CMD ["python", "main.py"]
```

- Docker Compose 실행

    ```
    docker-compose up -d
    ```

    - EC2에서 80포트 열어주기 

    - 크롬 브라우저를 통해, localhost:8080으로 접속이 가능하면 성공
    - main.py에서 3306포트와 mysqldb 호스트 이름을 사용한 것을 확인할 수 있음 

### dockerignore

- Dockerfile 작성시 COPY ./ /app 구문은 현재 폴더에 있는 모든 파일을 컨테이너 내의 /app 폴더로 복사

- 현재 폴더에는 Dockerfile도 있으며, 작업 환경에 따라서 예상치 못한 파일들이 있을 수 있음  예(vscode)

- COPY시 특정 파일/폴더는 제외하도록 현재 폴더에 .dockerignore 파일 작성 가능

    - gitignore와 동일한 방식

- .dockerignore 파일 포맷 

    ```
    #주석 
    */flask*
    */*/flask*
    flask?
    flask*
    ```

    - `*/flask*`: 현재 폴더의 어떤 하위 폴더든 (이것이 */가 의미하는 바) flask로 시작하는 폴더나 파일명은 제외

        - 예 : any/flask/kkk.txt, any/flask.txt

    - `*/*/flask*`: 현재 폴더의 하위 폴더의 하위폴더에서, flask로 시작하는 폴더나 파일명은 제외

        - 예: any/any/flask/kkk.txt, any/any/flask.txt
        - 하위 폴더의 깊이에 관계 없이 쓰려면, `**/flask*`와 같이 두개의 *로 작성하면 됨

    - `flask?`와 `flask*`: flask?의 물음표(?)는 한글자를 의미하므로, flaska나 flaskb와 같은 폴더 또는 파일을 의미하고, flask*는 flask로 시작하는 모든 폴더나 파일을 의미

    - 느낌표(!)를 쓰면,해당 조건은 **제외조건**에서 제외함을 의미함

        - 예 : 다음 예는 현재 폴더의 모든 .txt로 끝나는 파일을 제외하되, 단 flask.txt는 제외하지 말라는 뜻 

        ```
        *.txt
        !flask.txt
        ```

    

#### docker-compose logs

- 각 컨테이너의 모든 로그(출력결과) 확인

    ```
    docker-compose config
    ```

#### docker-compose config

- 실행 중인 Docker Compose의 docker-compose.yml 설정 확인

    ```
    docker-compose config
    ```

#### docker-compose exec 컨테이너 이름 명령

- 실행중인 컨테이너에 명령을 실행 

    ```
    docker-compose exec app /bin/bash
    ```

    
