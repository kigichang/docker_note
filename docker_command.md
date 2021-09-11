# Docker Command

## Docker 名詞解釋

1. **Image**: 預先準備的環境，有點像我們傳統的 ghost 好的 image 光碟片。
1. **Container**: 使用 image 啟動的執行環境，像是一個 vm 的環境，會執行預設的程式。
1. **Docker Hub**: 存放 images，官方有提供，類似 GitHub 的概念，可以共享 images，docker 預設會去官方的 Hub 找 images。公司內部也有自己架 Hub.
1. **Host**: 執行 docker 的主機。

以下 use case 都是使用 Linux like 環境。

## 基本指令

### Hello World

`docker run hello-world`

說明:

1. `run`: 啟動一個 image. **hello-world** 是 image 的名稱。docker 會自動判斷本地端有沒有該 image。如果沒有會自動下載。
1. Images 命名方式: **Repository:Tag**
    1. **Repository**: 命名方式 **[Domain_NAME_OR_IP:Port/]Path**，eg: `ubuntu:18.04` or `cicd.icu/cyberon/cr/run:0.0.1`。Domain_NAME_OR_IP 與 Port 可以省略，省略時 docker 會去找官方 Hub 找。如果自建 Hub Domain_NAME_OR_IP 會是 Hub 的 domain name or ip。
    1. **Tag**: 類似版本號，可省略。省略時，docker 會去找 **latest**。

### 查詢有那些 images

`docker images`

```shell
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        7 weeks ago         1.84kB
```

說明:

1. **REPOSITORY**: image 來自那個 resposoitory
1. **TAG**: image 的 tag
1. **IMAGE ID**: image 的 hex number 唯一值。

### 查詢目前有那些 container

`docker ps -a`

```shell
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
b19f70b2513a        hello-world         "/hello"            9 seconds ago       Exited (0) 8 seconds ago                       practical_bassi
```

說明:

1. `-a`: 是查全部的 container，包括已經停止的 container。如果省略 `-a`，只會列出正在執行中的 container。
1. **CONTAINER ID**: docker 會為每一個 container 取唯一的 hex number 當 ID。
1. **IMAGE**: 使用那個 image
1. **COMMAND**: container 預設執行的指令。
1. **PORTS**: container 有揭露那些網路 Port，這個欄位也會顯示 Host 主機的 Port 與 container Port 對應關係。
1. **NAMES**: container 別名，此名稱也不能重覆(唯一值)。如果沒有為 container 命名時，docker 會亂數取一個名字。

### 為 container 命名

命名方法有兩種:

1. 執行 `docker run` 時，加 `--name` 參數，eg：`docker run --name mytest1 hello-world`

    ```shell
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
    ceb40cda4540        hello-world         "/hello"            4 seconds ago       Exited (0) 3 seconds ago                       mytest1
    ```

1. 使用 `docker rename CONTAINER_ID_OR_NAME NEW_NAME` 為已存在的 container 重取名稱，eg: `docker rename mytest1 mytest2`

    ```shell
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
    ceb40cda4540        hello-world         "/hello"            46 seconds ago      Exited (0) 45 seconds ago                       mytest2
    ```

1. container name 是唯一值，不能重覆。

#### 為什麼要取名稱

1. 如果沒有取名稱的話，docker 會採亂數的方式來命名，管理上不方便。
1. 可以自動(程式化)佈署。

### 什麼都沒有的 ubuntu 環境

`docker run -it ubuntu:18.04`

說明:

1. `-it` 主要是由 `-i` 與 `-t` 兩者組成, 通常這兩個參數會一起使用。
    1. `-i`: 取得 container STDIN，之後可以與 docker 互動。
    1. `-t`: 配置偽終端。

### 背景執行

`docker run -d IMAGE [COMMAND]`, eg: `docker run -d ubuntu:18.04 tail -f /dev/null`

說明:

1. `-d`: Detached。讓 container 在背景執行。
1. 為什麼要加 `tail -f /dev/null`: 原本 **ubuntu:18.04** 預設指令是 **/bin/bash** 在背景執行時，會馬上結束，因此改執行 **tail -f /dev/null** 讓 container 不要馬上結束。

    ```shell
    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS                      PORTS               NAMES
    dd5268d8f3dd        ubuntu:18.04        "tail -f /dev/null"   4 seconds ago       Up 2 seconds                                    focused_robinson
    e205eaa2860a        ubuntu:18.04        "/bin/bash"           21 seconds ago      Exited (0) 19 seconds ago                       focused_ramanujan
    ```

### 查詢背景執行 container 的 STDOUT

`docker logs -f CONTAINER_ID_OR_NAME`

說明:

1. `-f`: 與 `tail -f` 效果類似，會等待程式的輸出結果。可省略。
1. `-t`: 可顯示每一則 log 的輸出時間。
1. 已經停止的 container 也可以使用。

### 執行 container 內的其他程式

`docker exec [-it] CONTAINER_ID_OR_NAME COMMAND [ARGS]`

1. `docker exec`: 執行在 container 內的某個應用程式。
1. `-it`: 同 `docker run`，如果執行的應用程式有需要從 STDIN 輸入資料或互動時，就要加 `-it`。

### 進入背景執行的 container

使用 `docker exec`, eg: `docker exec -it CONTAINER_ID_OR_NAME /bin/bash`

說明:

1. `-it`: 同 `docker run`
1. `/bin/bash`: 執行 linux 的 bash。

### 停掉 container

`docker stop CONTAINER_ID_OR_NAME`

### 啟動停止的 container

`docker start CONTAINER_ID_OR_NAME`

### 重啟 container

`docker restart CONTAINER_ID_OR_NAME`

### 刪除 container

`docker rm CONTAINER_ID_OR_NAME`

注意事項:

1. 在執行 `docker run` 時, 如果 container name 重覆，會執行失敗，此時就需要將舊的刪除
1. 如果要強制 container ，可加 `-f`. eg: `docker rm -f CONTAINER_ID_OR_NAME`
1. 無法直接刪除執行中的 container, 請加 `-f`。
1. 如果要刪除所有 container，可用 `docker rm -f $(docker ps -aq)`，請小心服用。

### 刪除 image

`docker rmi IMAGE_ID_OR_REPOSITORY_TAG`

注意事項:

1. 如果要強制刪除 image，請加 `-f`。eg: `docker rmi -f IMAGE_ID_OR_REPOSITORY_TAG`
1. 無法直接刪除執行中 container 使用的 image, 請加 `-f`.
1. 如果要刪除所有 images，可用 `docker rmi -f $(docker images -aq)`，請小心服用。

### Copy 檔案

docker 支援從 HOST 主機 copy 檔案進 container 或者 copy container 的檔案至 HOST.

1. Host to Container: `docker cp HOST_PATH_OR_FILE CONTAINER_ID_OR_NAME:PATH_OR_FILE`
1. Container to HOST: `docker cp CONTAINER_ID_OR_NAME:PATH_OR_FILE HOST_PATH_OR_FILE`

注意事項:

1. `docker cp` 支搜 copy 整個目錄。
1. container 在停止時，依然可以 copy 資料進 container.

### 備份完整的 container

docker 支援 將 container 的檔案與設定儲成一個新的 image.

`docker commit -m COMMIT_MESSAGE -a AUTHOR CONTAINER_ID_OR_NAME [REPOSITORY[:TAG]]`

說明:

1. `-m`: commit 註解，可省略
1. `-a`: 作者資訊，可省略
1. `[REPOSITORY[:TAG]]`: image 的 respository 與 tag, 雖然可以省略，但建議還是自行命名。

### 將 image 匯出成檔案

`docker save -o DEST_FILE_NAME IMAGE_ID_OR_RESPOSITORY_TAG`

說明:

1. `-o`: 請加 `-o` 匯出成檔案

### 將完整的 image file 匯入 docker

`docker load -i SRC_FILE`

1. `-i`: 來源檔案

### 只匯出 container file system 資料

`docker export -o DEST_FILE CONTAINER_ID_OR_NAME`

注意事項：

1. 此指令只會匯出當下 container 的 file system 的資料，並沒有原先 image 的設定。

### 只匯入 images 檔案中的 file system 進 docker

`docker import SRC_FILE [REPOSITORY[:TAG]]`

1. 此指令只會入出 image 檔案中的 file system 的資料，並沒有原先 image 的設定與 container 狀態。

### commit/load, export/import

指令  | 匯出 | 匯入指令 |
- | - | - |
commit | container 設定與檔案 | load or import |
export | container 檔案 | import only |

指令  | 檔案來源 | 匯入 |
- | - | - |
load | commit | containter 設定與檔案
import | commit or export | container 檔案

## 進階操作

### Container 執行完畢後，自動刪除

加 `--rm` 參數。eg: `docker run --rm --name mytest4 ubuntu:18.04 echo 'hello'`

### 新增或覆寫 container 的環境變數

加 `-e` 參數。eg: `docker run -d -e MY_CODE=123 --name mytest5 ubuntu:18.04 tail -f /dev/null`

### 在 HOST 主機重啟後，自動重啟 container

在執行 `docker run` 時，加 `--restart always`, eg: `docker run -d --restart always ubuntu:18.04 tail -f /dev/null`。如此 container 當 Host 重啟後，會自動重啟。

注意事項:

1. 在安裝完 docker 後，記得將 docker 註冊成 service，如此主機重啟時，才會自動啟動 docker。docker 才會去執行既有的 container。
1. 在 `docker run`, 使用 `--restart always`，如果 container 執行失敗，會一直自動重啟，此時可以使用 `docker stop CONTAINER_ID_OR_NAME` 來停掉 container。

### Mount Host 目錄至 container

在執行 docker run 時，加 `-v HOST_SRC_PATH:CONTAINER_DEST_PATH`. eg: ```docker run -d --name mytest -v `pwd`/mytest:/mytest ubuntu:18.04 tail -f /dev/null```

說明:

1. ``` `pwd` ```: 中的 `pwd` 是 shell 指令
1. HOST_SRC_PATH: 必須是完整的絕對路徑，不能是相對路徑。
1. 使用 `-v` 時，如果 container 內的目錄資料，會被 Host 取代，並不會匯出至 Host 目錄上。
1. 如果有多個目錄要 mount 時，可以下多個 `-v`。

注意事項:

1. 不論是用 export or commit 備份 container，都不會把設定在 **volume** or 用 **`-v`** mount 的目錄匯出。使用時要特別小心
1. 原則上不要把檔案儲在 container 內，都儲在 mount 的目錄下，之後搬移比較方便，也避免 container crash 時，檔案也都不在。

### 將 Host 主機的 port 對應至 container port

在執行 docker run 時，加 `-p HOST_PORT:CONTAINER_PORT`。eg: `docker run -d --name mynginx -p 8080:80 nginx:latest`

```shell
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
c86a217193fb        nginx:latest        "nginx -g 'daemon of…"   2 seconds ago       Up 1 second         0.0.0.0:8080->80/tcp   mynginx
```

注意事項:

1. `HOST_PORT` 也可以指定 ip, eg: `-p 127.0.0.1:8080:80`，只 bind 127.0.0.1:8080 至 container 的 80 port.
1. 如果有多個 port 要做對應時，可以下多個 `-p`, eg: `-p 80:80 -p 443:443`

### 連結兩個 container

通常用在兩個 container 是類似 client/server 或者 application/database 關係，主要是要讓某個 container 可以連線至另一個 container。

在執行時，加入 `--link SRC_CONTAINER_ID_OR_NAME:DEST_NAME`, eg: `docker run -d --name mytest2 --link mytest1:mytest ubuntu:18.04 tail -f /dev/null`

```shell
#docker ps -a
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS               NAMES
323e950101ea        ubuntu:18.04        "tail -f /dev/null"   3 seconds ago       Up 2 seconds                            mytest2
0ec5717e7fea        ubuntu:18.04        "tail -f /dev/null"   36 seconds ago      Up 35 seconds                           mytest1
```

```shell
#docker exec mytest2 cat /etc/hosts
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2  mytest 0ec5717e7fea mytest1
172.17.0.3  323e950101ea
```

注意事項：

1. source container 必須是在執行中的狀態。
1. 如果要 link 多個 container 時，可下多個 `--link`。

## 其他指令

### help

`docker help` or `docker help COMMAND`, eg: `docker help cp`

### 查詢某個 container 執行狀態

`docker top CONTAINER_ID_OR_NAME`

### 查詢目前所有 container 消秏資料狀況

`docker stats`

### 查詢目前 docker service 狀態

`docker info`

### 查詢 images or container 底層資料

查詢 images 或 container 底層 docker 物件資料。

- `docker inspect CONTAIN_ID_OR_NAMAE`
- `docker inspect IMAGE_ID_OR_REPOSITORY_TAG`

### 強制停止 container (send KILL signal to container)

`docker kill CONTAINER_ID_OR_NAME`

### 為 image 新加 repository:tag

`docker tag SRC_IMAGE_ID_OR_REPOSITORY_TAG TARGET_REPOSITORY_TAG`, eg: `docker tag ubuntu:18.04 myubuntu:18.04`

### 下載 image

雖然在 `docker run` 時，會自動下載必要的 images, 但也可以手動自行下載。

`docker pull REPOSITORY[:TAG]`, eg: `docker pull ubuntu:18.04` or `docker pull cicd.icu/cyberon/cr/lib:0.0.1`

### Login/Logout

用在登入官方或私有的 docker hub

- `docker login [DOMAIN]`, eg: `docker login` or `docker login cicd.icu`
- `docker logout [DOMAIN]`, eg: `docker logout` or `docker logout cicd.icu`
