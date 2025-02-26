## main

EXPOSE 80 : 监听80，暴露80 ，两个作用

### 1、将变量传递到dockerfile中

docker build --build-arg BUILD_ENV=some_value

```
FROM golang:1.18.4 AS llm_builder

ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64 \
    GOPROXY=https://goproxy.cn,direct

WORKDIR /app
COPY . .
RUN go mod tidy
RUN go build -o lanyan-llm-service .

FROM alpine:3.18

RUN apk update
RUN apk add git

ENV TZ Asia/Shanghai
RUN apk add tzdata

COPY --from=llm_builder /app/lanyan-llm-service /lanyan-llm-service
#COPY --from=llm_builder /app/.yaml /.yaml

# 设置环境变量
ARG BUILD_ENV
ARG GIT_USERNAME
ARG GIT_PASSWORD

# 克隆配置代码仓库并将文件复制到容器中
RUN git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@codeup.aliyun.com/donglan/lanyan/backend/lanyan-go-conf.git
RUN cp lanyan-go-conf/${BUILD_ENV}/llm-service/app.yaml /.yaml

EXPOSE 19121
ENTRYPOINT ["./lanyan-llm-service", "llm"]
```

### 2、执行错误了继续执行下去

可以使用`|| true`或者`&& echo "Command failed but I'm ignoring it"`来忽略错误。

```
docker stop XXX || true
docker stop XXX || echo "container not exist"
```

### 3、打包

```
docker build -f DockerfileForVideoService_CPU  -t my-task .    
```





