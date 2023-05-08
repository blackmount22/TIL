FROM 명령어, 즉 node 이미지를 개조해서 만드는 이미지

```jsx
FROM node:12.18.4
```

RUN 명령어는 이미지를 생성하는 과정에서 실행할 명령어

```jsx
# 이미지 생성 과정에서 실행할 명령어
RUN npm install -g http-server
```

WORKDIR 명령어는 이 안에서 명령어를 실행할 위치를 설정

```jsx
#이미지 내에서 명령어를 실행할(현 위치로 잡음) 디렉토리 설정
WORKDIR /home/node/app
```

```jsx
# 컨테이너 실행시 실행할 명령어
CMD ["http-server", "-p", "8080", "./public"]
```

```jsx
#이미지 생성 명령어 (현 파일과 같은 디렉토리에서)
#docker build -t {이미지명}

#컨테이너 생성 & 실행 명령어
#docker run --name {컨테이너명} -v ${pwd}:/home/node/app -p 8080:8080 {이미지명}
```

---
