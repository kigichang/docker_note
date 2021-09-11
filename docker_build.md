# Docker Image

名詞解釋：

1. Dockerfile: 製作 image 的描敘檔
1. Context Path: 放製作 image 所需資料的目錄

## 自製 image 建議方式

1. 建立目錄，放 image 會用到的資料，可以是專案的目錄。
    1. 在該目錄下，可以編寫 **.dockerignore**。因為 docker 在製作 image，會掃描目錄，加 **.dockerignore** 可以加速掃描，也可以避免 copy 不要的檔案進 image.
1. 撰寫 docker file. 請取檔名為 **Dockerfile**。
    1. docker 預設會找 **Dockerfile**，當然也可以不用取這個檔名，但在 build image 時，就需要加 `-f` 指定檔名。

## 指令

`docker build -t RESPOSITORY:TAG .`

說明：

1. `-t` 為 image 命名，管理方便，也可以 push 至 docker hub.
1. 請切換目錄至製作 image 的目錄
1. 執行 `docker build .`, docker 會找該目錄下的 **Dockerfile**。
1. 如果不要切換目錄，則可以用 `docker build -f FILE_NAME -t RESPOSITORY:TAG CONTEXT_PATH`, eg: `docker build -f docker_build/Dockerfile -t cicd.icu/kigi/docker_build/mybuild:0.0.1 docker_build`

## Dockerfile

Dockerfile 是敘述如何製作 image，主要會有：

1. 使用那個 image 當作基礎
1. 執行安裝套件指令，或其他建立執行環境的指令，eg: `mkdir`
1. copy 檔案
1. 設定環境變數 (非必要)
1. 設定揭露網路 port (非必要)
1. 設定可讓 HOST mount 目錄 (非必要)，如果有設定，當匯出 container 時，該目錄下的檔案不會被匯出。
1. 設定預設執行程式。

### Sample

DockerFile.

```Dockerfile
FROM ubuntu:18.04

LABEL maintainer="kigi.chang@gmail.com"
RUN apt-get update && apt-get install -y vim
RUN rm -rf /var/lib/apt/lists/*
RUN mkdir /home/mytest && mkdir /home/mytest/a && mkdir /home/mytest/b
WORKDIR /home/mytest
COPY a.txt /home/mytest/a/
COPY b.txt b/
ADD data/ /home/mytest/c/
ADD data/mytar.tar.gz /home/mytest/
COPY . /home/mytest/
EXPOSE 8080
VOLUME [/home/mytest/c]
ARG CODE=hello
ENV MY_CODE=${CODE}
ENTRYPOINT ["./entrypoint.sh"]
CMD ["hello", "world"]
```

### Dockerfile 指令說明

1. `FROM`: 使用那個 image 作基礎。
1. `LABEL`: 撰寫 image 的 meta data. 常用來撰寫作者是誰。eg: `LABEL maintainer="kigi.chang@gmail.com"`。
1. `RUN`: 執行指令, eg: `apt-get`, `rm`, or `mkdir`。
1. `WORKDIR`: 設定工作目錄，此工作目錄是指 container 的目錄。
1. `EXPOSE`: 揭露那些網路 port。
1. `ARG`: 設定 build image 時的變數。
1. `ENV`: 設定 container 執行時的環境變數。之後可使用 `docker run -e` 來覆寫。
1. copy 檔案進 image 的指令有兩種: `COPY` and `ADD`。兩者的用法都是 `COPY SRC_PATH_OR_FILE DEST_PATH_OR_FILE` 與 `ADD SRC_PATH_OR_FILE DEST_PATH_OR_FILE`。
    1. 都支援 copy 目錄至 image。
    1. 來源都是以 Context Path 為基準的相對路徑。
    1. 如果是 copy 至 image 目錄時，在目錄名稱結尾請加 **`/`**，eg: `/home/mytest/`, `ADD` 一定要加，但 `COPY` 可不用；為了必免混餚與增加可讀性，如果是目錄還是請加 `/`。
    1. 如果 image 的目錄是相對路徑時，以 `WORKDIR` 為基準的相對路徑, eg: `b/` 是指 `/home/mytest/b/`。
    1. `ADD` 的來源可以是網址或 gzip 的檔案，如果是 gzip 檔案，則會自動解壓縮。
1. `VOLUME`: 在生成 Container 時，自動在 HOST 主機上，產生 volume，`VOLUME` 的使用，可以看 docker command 說明。
1. 設定預設執行的程式有兩種：`ENTRYPOINT` and `CMD`。
    1. 主要設定是 `ENTRYPOINT`，如果有設定 `ENTRYPOINT`，則 `CMD` 當作 `ENTRYPOINT` 的參數。
    1. 如果沒有 `ENTRYPOINT` 則會用 `FROM` image 的 `ENTRYPOINT`
    1. `ENTRYPOINT` 支援 shell script。
    1. 如果設定 `CMD`，則是覆寫掉 `FROM` image 的 `CMD`

## Makefile

Makefile.

可以搭配 **Makefile** 讓製作過程更簡便。

```Makefile
.PHONY: clean code

IMAGE_NAME=cicd.icu/kigi/docker_build/mybuild
VERSION=0.0.1

all:
	docker build --pull --no-cache -t $(IMAGE_NAME):$(VERSION) .	

code:
	docker build --pull --no-cache --build-arg CODE=cyberon -t $(IMAGE_NAME):$(VERSION) .

clean:
	- docker rmi -f $(IMAGE_NAME):$(VERSION)

test:
	- docker stop mytest
	- docker rm mytest
	- docker run -d --name mytest $(IMAGE_NAME):$(VERSION)

push:
	make all
	docker push $(IMAGE_NAME):$(VERSION)
```

### docker build 參數說明

1. `--no-cache`: build image 時，不要用先前的 cache 檔。
1. `--build-arg`: 取代 Dockerfile 內 `ARG` 的變數
1. `--pull`: 每次都去檢查 `FROM` 的 image.

## Push

`docker push REPOSITORY:TAG`

說明：

1. 自家的 docker hub 是使用自家 gitlab 架設，登入時，請用自家 gitlab 的帳號、密碼登入
1. 如果要 push image 時，請先登入。官方：`docker login`, 自家：`docker login cicd.icu`。
1. 如果需要與其他人共享 image，請申請自家的 gitlab 帳號。
1. 自家 image 的命名方式，是以在 gitlab 上專案為基礎，因此 REPOSITORY 名稱會是 gitlab 專案的 url。eg: `cicd.icu/kigi/docker_build`
1. 如果該專案有多個 image 檔時，可以再往下加 PATH_NAME。eg: `cicd.icu/kigi/docker_build/mybuild`
1. docker images 放在 gitlab 專案頁面，左側的 **Registry**。eg: [https://cicd.icu/kigi/docker_build/container_registry](https://cicd.icu/kigi/docker_build/container_registry)
1. 登出: `docker logout` or `docker logout cicd.icu`