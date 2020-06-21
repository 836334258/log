```dockerfile
FORM
RUN
ENV param1 /tmp
WORKDIR $param1
ADD . $param1
LABEL
EXPOSE
VOLUME
COPY
ARG  docker build --build-arg lang=php -t king/test:1.0.0
SHELL 切换shell

```

- build命令`docker build -t king/test:1.0.0 .`

- dockerfile

  ```dockerfile
  FROM nginx:latest
  LABEL author="king" email=836334258@qq.com
  ENV MY_DIR /king/www
  ENV MY_DIR2 /etc/nginx
  VOLUME ${MY_DIR}
  VOLUME ${MY_DIR2}
  RUN echo '开始任务...'
  RUN chmod 777 -R ${MY_DIR}
  ADD *.html $MY_DIR;
  COPY *.conf $MY_DIR2
  WORKDIR ${MY_DIR}
  EXPOSE 83
  ARG lang=java
  RUN echo ${lang}
  RUN echo '结束任务...'
  ```

  