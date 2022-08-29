# 컨테이너 활용과 연결

## Docker로 jupyter notebook 띄우기

- jupyter notebook은 파이썬 코드를 실행하기 위한 툴



- 컨테이너 내부에서 jupyternotebook이 실행되는 폴더인 /home/jovyan 폴더를 호스트 PC의 현재 폴더로 만들어서, 호스트 PC에서 docker를 실행하는 폴더에 있는 주피터 노트북 파일 작업이 가능하도록 함



```
docker run --rm -d -p 8888:8888 -v /home/ubuntu/2021_LEARN:/home/jovyan/work jupyter/datascience-notebook
```

