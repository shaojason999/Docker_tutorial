# Docker_tutorial

* 在本來的OS上指令開頭標記為 $
* Container中的指令開頭標記為 #

### Docker安裝教學
以下例子以在Ubuntu上安裝docker做說明，並且示範lamp的部屬
1. 快速安裝
```
$ curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```
2. 啟動
```
$ sudo service docker start
```
3. 下載所需image:
* 從docker hub下載linode的lamp
    * 預設從docker hub下載
```
$ docker pull linode/lamp
```
4. 查看是否下載成功
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
linode/lamp         latest              2359fa12fded        4 years ago         372MB
```
5. 執行images(創建container)
```
$ sudo docker run -t -i linode/lamp:latest /bin/bash
```
* :latest可以不用打(預設tag是latest)

### 常用指令
1. 下載image
```
$ docker pull repository:tag
```
* 注意tag(不打的話是latest，但如果沒有latest版本，則無法pull)

2. 執行某個image
```
$ sudo docker run -t -i --name NAME repository:tag /bin/bash
```
* -i -t 是為了讓我們進到container的shell下指令
3. 離開container
```
root@0f00579f5905:/# exit
```
* 或是在container按Ctrl+P, Ctrl+Q，則會退出container但不會停止運作(一般exit會停止運作)
4. 產生一個新的image(另一個方法，下一段會教)
* exit某個container後，下次再次開啟image則又是產生一個新的container，所以如果要紀錄某個container的話，就要把container變成image
```
$ sudo docker commit -m "Comment here" 0f00579f5905 test1:v2 
```
* 那串ID來自container中的`root@0f00579f5905:/#`這裡

5. 查看docker image
* TAG是用來辨識同一個REPOSITORY中的不同image
* IMAGE ID是唯一的，若一樣表示是同一個image
```
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test1               v2                  722421f22f8d        5 minutes ago       431MB
test                v1                  a6dd71ed5bbe        About an hour ago   431MB
linode/lamp         latest              2359fa12fded        4 years ago         372MB
```

6. 查看docker訊息
```
$ docker info
```
7. 查看container
* 正在run的container
```
$ docker ps
```
* 全部container
    
```
$ docker ps -a
```
* 最新創建的30個container
```
$ docker ps -n 30
```
8. 停止container
```
$ docker stop <Container_ID>
```
9. 刪除conatin及image(刪image前要先刪container)
```
$ docker rm <Container_ID>
$ docker rmi <REPOSITORY:TAG>
```
10. 回到container裡面
* 如果container的status是exit的要先start
```
$ docker start <C_ID>
//或是直接用下面的2.來做到start+進去
```

* 以下分幾種方式進去
```
//1.
$ docker attach <C_ID>
//2.
$ docker start -it <C_ID>
//3.
$ docker exec -it <C_ID> bash
```
* 1.跟2.相同，進去container後如果打exit離開，則container會停止運作
* 3.進去後打exit只會離開並不會停止

11. Docker Hub login
```
$ docker login
```
12. Push to Docker Hub
```
$ docker push shaojason999/push_test
The push refers to repository [docker.io/shaojason999/push_test]
67720bcd7e89: Pushed 
8f42a46398d5: Pushed 
1ecc4f0639b8: Pushed 
ed7fe540ea27: Mounted from linode/lamp 
af3341e56051: Mounted from linode/lamp 
5f70bf18a086: Mounted from linode/lamp 
541bc77c741d: Mounted from linode/lamp 
bf721bc7e47e: Mounted from linode/lamp 
db3e51bbcde6: Mounted from linode/lamp 
v1: digest: sha256:0f4df2423300f10a9b872f364522a0642b6c99aa5208bdbf56d066f34620e14f size: 2613
```
* 因為是是私人空間(非官方)，所以要加username
* image名稱也要是username/image_name
* image base不用上傳(mounted from xxx/xxx)
![](https://i.imgur.com/bMBsysc.png)


### Image製作
#### 方法1
把修改過的container存成新的image
```
$ sudo docker commit -m "Comment here" 0f00579f5905 test1:v2 
```
#### 方法2
利用Dockerfile創建新的image
* image是一層一層疊上去的

假設一個情境: 我在A主機的環境想複製到B主機使用(A,B都有Docker)
1. 在A的某個資料夾下有 index.js package.json 兩個檔案
2. 在同資料夾下創建一個名叫Dockerfile的檔案，內容為
```
FROM node:9.2.0

COPY index.js package.json /app/

WORKDIR /app

RUN npm install && npm cache clean --force

CMD node index.js

# this is a comment
```
* FROM的意思是從docker hub下載node的image做為base image
* COPY 為複製檔案
* WORKDIR 切換目錄
* RUN 要在base image上跑甚麼指令(堆疊image上去)
* CMD 為做好的image跑起來時的預設指令


![](https://i.imgur.com/KtCdTov.png)

3. Build
```
$ docker build -t image_name .
Sending build context to Docker daemon  114.7kB
Step 1/5 : FROM node:9.2.0
 ---> c1d02ac1d9b4
.
.
.
Successfully built c1d02ac1d9b4
Successfully tagged test5:latest
```
* .是Dockfile的路徑位置
* 每一步都建立了一個新的容器，在容器中執行指令並提交修改（就跟前面的 docker commit 一樣）
* 當所有的指令都執行完畢之後，返回了最終的image id。所有的中間步驟所產生的容器都會被刪除和清理

4. 上傳image到docker hub

或是其他方法:

3. 把Dockerfile, index.js, package.json三個檔案上傳到github之類的地方，再從B主機下載
4. 在B主機執行Build

#### 方法3
一樣是Dockerfile，但是是利用工具自動從image產生
[參考資料](https://philipzheng.gitbooks.io/docker_practice/content/dockerfile/file_from_image.html)

### 觀念釐清
1. Dockerfile的base image是甚麼都可以，可以是最基本的ubuntu，然後再在base image上安裝其他套件
2. Dockerfile不是直接把A電腦的環境複製給B用，而是需要先知道A的環境，然後打到Dockerfile裡(除非使用方法3的工具)

---
參考資料  
[Docker 實戰系列（一）：一步一步帶你 dockerize 你的應用](https://larrylu.blog/step-by-step-dockerize-your-app-ecd8940696f4)  
[Docker —— 從入門到實踐](https://philipzheng.gitbooks.io/docker_practice/content/)
