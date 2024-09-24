# 🚀 Docker-Image-Optimization 
Docker 이미지를 최적화하면 효율적인 리소스 활용, 빠른 배포, 보안성 강화를 기대할 수 있다.

## 🛠️ Docker 이미지를 최적화하기 위한 팁을 다룬다.

### 1. 🐳 사이즈가 작은 기본 이미지 선택 
   - Alpine Linux나 Scratch 같은 최소한의 크기를 갖는 기본 이미지로 시작하기.
   - 이러한 이미지들은 경량화 되어있고, 필수 구성 요소만 포함하고 있어 이미지 크기와 공격자가 악용할 수 있는 취약점을 줄일 수 있다.

```dockerfile
FROM nginx:alpine
```

### 2. 1️⃣ 단일 책임 원칙
   - Docker 이미지는 단일 책임을 갖도록 하기
     각 서비스에 대해 별도의 이미지를 사용하고 Docker Compose나 Kubernetes를 사용하여 구성하는게 좋다.

### 3. 🔄 빌드 과정을 여러 단계로 나누기
   - 하나의 Dockerfile에서 여러 개의 FROM 문을 사용할 수 있다.
     최종 이미지에서 빌드 시간 종속성과 불필요한 파일을 제거하면 이미지 크기를 줄이는 데 도움이 된다.

```
# Build stage
FROM node:14-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:14-alpine as production

WORKDIR /app
COPY --from=build /app/package*.json ./
RUN npm ci --production
COPY --from=build /app/dist ./dist

CMD ["npm", "start"]
```

### 4. 📉 레이어 최소화
   - 여러 명령을 하나의 RUN 지시문으로 결합하여 Docker 이미지의 레이어 수를 줄이기
   이미지 크기를 줄이고 빌드 프로세스를 가속화할 수 있다.
       
```
RUN apt-get update && \\
apt-get install -y git && \\
apt-get clean && \\
rm -rf /var/lib/apt/lists/*
```

### 5. 🚫 .dockerignore 사용
   - .dockerignore 파일을 만들어 Docker 빌드 컨텍스트에서 불필요한 파일과 디렉토리를 제외하기
    빌드 시간을 줄이고 큰 파일이 이미지에 포함되는 것을 방지

### 6. 🔖 특정 버전 태그 사용
  - latest 태그 사용은 편리하지만 예상치 못한 변경으로 인한 이슈가 발생할 수 있다.
    - 특정 버전에 대한 태그를 사용하여 안정성을 보장하는 것이 좋다.
```
FROM nginx:&lt;tag&gt;
```
### 7. 📝 docker file 지시문 최적화
  - docker file의 명령어를 패키지 설치에 특정 버전을 사용하고, 종속성의 수를 최소화, 설치 후 불필요한 패키지를 제거하도록 구성한다.
     
```

# 환경변수를 통한 반복 코드 제거
ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . $APP_HOME

-------------------------------------

# 패키지 설치에 특정 버전을 사용
RUN apt-get install -y package=1.0.0

-------------------------------------

# 불필요한 패키지 제거
RUN apt-get update && \\
apt-get install -y git && \\
apt-get clean && \\
rm -rf /var/lib/apt/lists/*

```

### 8. 📦 아티팩트 압축
  - 애플리케이션이 컴파일된 바이너리나 정적파일과 같은 빌드 아티팩트를 생성하는 경우 이를 Docker이미지에 복사하기 전에 압축하기.
    - 이미지 크기를 줄이고 빌드 프로세스를 가속화 한다.
         
### 9. 🔍 이미지 레이어 검사
  - `docker history`와 `docker inspect`와 같은 도구를 사용하여 이미지 레이어를 분석한다.
    - Dockerfile에서 불필요한 파일과 명령을 제거할 수 있도록 최적화 할 곳을 찾아보기
         
### 10. 🧹 Docker 이미지 정리하기
   - docker system prune 명령을 사용하여 사용하지 않는 이미지, 컨테이너, 볼륨, 네트워크 등을 정리하여 디스크 리소스를 확보하고 성능         을 향샹 시킨다.
     
### 11. ♻️ 캐싱 구현
   - Docker의 빌드 캐시를 활용하기 위해 Dockerfile을 캐시 활용을 최대화 하는 방식으로 구성한다.
     - 자주 변경되는 지시문을 Dockerfile의 끝 부분에 배치하여 캐시 무효화를 최소화 하기
       - Docker는 각 명령어를 실행할 때마다 그 결과를 캐시에 저장한다. 만약 Dockerfile의 명령어 중 하나라도 변경되면 그 명령어와 그             이후의 모든 명령어의 캐시가 무효화된다.
       - Docker는 각 명령어(지시문)를 실행할 때마다 캐시를 사용하여 이전 단계의 결과를 재사용한다.
       - 자주 변경되는 지시문이 Dockerfile의 위쪽에 있으면, 그 명령어가 변경될 때마다 그 이후의 모든 명령어도 다시 실행되어야 하므              로 비효율적이다.
### 12. 🔒 보안 스캔
   - Docker 보안 스캔 도구를 활용하여 보안 취약점을 식별, 수정
     - 정기적으로 이미지의 취약점을 스캔하고 필요에 따라 보안 패치를 적용하기.
       
### 13. 🐜 더 작은 라이브러리 사용
   - 완전한 배포판 대신 BusyBox나 Microcontainers 사용을 고려

### 14. 🧽 설치 후 정리
   - 패키지 설치중 생성된 임시 파일과 캐시를 제거하여 이미지 크기 줄이기

### 15. 🧊 Docker Squash 사용
   - Docker Squash를 사용하여 레이어를 병합함으로써 Docker이미지의 크기를 줄일 수 있다.
     - 그러나 빌드 시간이 증가하고 캐시 가능성이 감소할 수 있으므로 주의가 필요하다.


## 실습

### 1. 기본이미지 비교 실습
**- stable버전 이미지**
```
# nginx:stable 사용
FROM nginx:stable

COPY . /usr/share/nginx/html
```

```
#캐시의 변수를 제외하고 단순 빌드 속도를 측정하기 위한 명령
time docker build --no-cache -t my-nginx-1 -f Dockerfile .
```
![image](https://github.com/user-attachments/assets/ce8abc5f-7c5c-4a04-ab40-c2d40ff7c750)
![image](https://github.com/user-attachments/assets/a18bf4d1-4334-442f-9fbb-d3c64666926c)



**- alpine버전 이미지**
```
# nginx:alpine 사용
FROM nginx:alpine

COPY . /usr/share/nginx/html
```


```
time docker build --no-cache -t my-nginx-2 -f Dockerfile .
```
![image](https://github.com/user-attachments/assets/3880d71c-896f-4590-bacd-d0979ef512c6)
![image](https://github.com/user-attachments/assets/922b90f2-4878-4e0c-bd28-88ae6efe89d8)


size와 빌드 속도 모두 alpine 버전이 우세하다.



### 2. 멀티스테이지 빌드 실습

       

