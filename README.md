# Federated_Download
This repository stored the source code for downloading the result of ferderated learning.
系列文章：这个系列已完结，如对您有帮助，求点赞收藏评论。
读者寄语：`再小的帆，也能远航！`

  1. [【k8s完整实战教程0】前言](https://blog.csdn.net/qq_47058489/article/details/130152468)
 2. [【k8s完整实战教程1】源码管理-Coding](https://blog.csdn.net/qq_47058489/article/details/130152543)
 3. [【k8s完整实战教程2】腾讯云搭建k8s托管集群](https://blog.csdn.net/qq_47058489/article/details/130153427)
 4. [【k8s完整实战教程3】k8s集群部署kubesphere](https://blog.csdn.net/qq_47058489/article/details/130153492)
 5. [【k8s完整实战教程4】使用kubesphere部署项目到k8s](https://blog.csdn.net/qq_47058489/article/details/130153784)
 6. [【k8s完整实战教程5】网络服务配置（nodeport/loadbalancer/ingress）](https://blog.csdn.net/qq_47058489/article/details/130153925)
 7. [【k8s完整实战教程6】完整实践-部署一个federated_download项目](https://blog.csdn.net/qq_47058489/article/details/130154196)

---
# 1 Coding代码仓库开发源码
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4bdd6a5919d4c228a2feb7173b96d42.png)

# 2 本地测试
## 2.1 git拉取到本地仓库

```bash
17211@hqc MINGW64 ~
$ cd /d/research/git_repository/

17211@hqc MINGW64 /d/research/git_repository (master)
$ ls
Federated/  federated_with_flask/  hello_world/

17211@hqc MINGW64 /d/research/git_repository (master)
$ git clone https://e.coding.net/hqc12/hqc/federated_download.git
Cloning into 'federated_download'...
remote: Enumerating objects: 31, done.
remote: Counting objects: 100% (31/31), done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 31 (delta 5), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (31/31), 12.44 KiB | 849.00 KiB/s, done.
Resolving deltas: 100% (5/5), done.

17211@hqc MINGW64 /d/research/git_repository (master)
$ ls
Federated/  federated_download/  federated_with_flask/  hello_world/

17211@hqc MINGW64 /d/research/git_repository (master)
$ cd federated_download/

17211@hqc MINGW64 /d/research/git_repository/federated_download (master)
$ ls
Dockerfile  README.md  main.py  requirement.txt  static/  templates/
```

## 2.2 本地运行测试
打开pycharm，运行测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/fa47300a17154aecadfe492338221ea0.png)

出错了，查找资料好像说是conda环境中存在两个该.dll文件造成的，运行在docker中应该不会出现这个问题。
为了进行本地测试，还是在本地解决一下，代码加上：

```bash
import os
os.environ["KMP_DUPLICATE_LIB_OK"]="TRUE"
```

再次运行测试：
![在这里插入图片描述](https://img-blog.csdnimg.cn/241c6936d2f34b109a60f868b3c40c55.png)

## 2.3 访问
两个网址都可以进行访问
点击即可下载
![在这里插入图片描述](https://img-blog.csdnimg.cn/aea7dc16546c4034bd0d028102b8ccaa.png)

成功！！！
## 2.4 有个小问题
好像没法指定下载到设置的downloads文件夹中，就是直接浏览器下载。因此这个文件夹可以不要了。
# 3 制作成容器镜像
## 3.1 创建制品仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/6dd72a7d928242159b41bc42ef77d627.png)

直接使用现有仓库
## 3.2 构建计划
采用之前的构建计划，更换一下代码仓库即可
另外，为提升二次构建镜像的速度，设置缓存目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/30d8d5d609b5453a9d16fa3289b1417a.png)

缓存目录设置参考：https://coding.net/help/docs/ci/configuration/cache.html
由于使用的是pip下载依赖，因此设置对应的缓存目录
## 3.3 立即构建
构建时 Dockerfile中COPY源码进容器这一步骤 出现问题，多方尝试之后，下面这样是ok的。

```bash
FROM python:3.7
WORKDIR . # 指定容器工作目录
COPY Dockerfile main.py requirement.txt ./ 
COPY /templates/download.html ./templates/download.html
RUN pip install -r requirement.txt
EXPOSE 5000
RUN /bin/bash -c 'echo init ok'
CMD ["python", "main.py"]
```

注意：
1. 目标路径必须是一个目录，并以/ 结尾
2. 不能直接复制文件夹，它默认会只复制文件夹中的文件，破坏文件结构
构建成功：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f47802305ed2494aa66e6035056eb6f9.png)

## 3.4 镜像仓库查看
成功！
![在这里插入图片描述](https://img-blog.csdnimg.cn/8206e87f84d44eafa254a45484b8f127.png)

# 4 搭建k8s集群
按之前的记录搭建即可
# 5 docker镜像测试
## 5.1 本地拉取镜像

```bash
# 登录
ubuntu@VM-1-15-ubuntu:~$ sudo docker login -u federated_project-1667453994942 -p 55a845e185c9bda4ba21fd2227a7538b55717b00 hqc12-docker.pkg.coding.net
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
# 拉取
ubuntu@VM-1-15-ubuntu:~$ sudo docker pull hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download:v0.0.1
v0.0.1: Pulling from hqc/federated_project/federated-download
17c9e6141fdb: Pull complete 
de4a4c6caea8: Pull complete 
4edced8587e6: Pull complete 
a7969cffbf46: Pull complete 
74fbfde6af91: Pull complete 
16fe51aed899: Pull complete 
a418194ab798: Pull complete 
e1b9101d5fa4: Pull complete 
c0b070a4672c: Pull complete 
7094a060c489: Pull complete 
6927575c3e2a: Pull complete 
be9ca32391c3: Pull complete 
aa5d393447e7: Pull complete 
Digest: sha256:4a2cb2995c0a9e10e2bef840a39e96843c043d50256c70d95bd7fc2f2a0362fe
Status: Downloaded newer image for hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download:v0.0.1
hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download:v0.0.1
```

## 5.2 本地运行镜像

```bash
# 查看当前镜像
ubuntu@VM-1-15-ubuntu:~$ sudo docker images
REPOSITORY                                                             TAG                 IMAGE ID            CREATED             SIZE
hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download   v0.0.1              135908d3f14b        50 minutes ago      3.32GB
...

# 打新的标签
ubuntu@VM-1-15-ubuntu:~$ sudo docker tag 135908d3f14b federated-download:v0.0.1
...

# 删除之前的镜像
ubuntu@VM-1-15-ubuntu:~$ sudo docker rmi hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download:v0.0.1
Untagged: hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download:v0.0.1
Untagged: hqc12-docker.pkg.coding.net/hqc/federated_project/federated-download@sha256:4a2cb2995c0a9e10e2bef840a39e96843c043d50256c70d95bd7fc2f2a0362fe

# 再次查看
ubuntu@VM-1-15-ubuntu:~$ sudo docker images
REPOSITORY                                                   TAG                 IMAGE ID            CREATED             SIZE
federated-download                                           v0.0.1              135908d3f14b        51 minutes ago      3.32GB
...

```
## 5.3 本地直接运行测试镜像
```bash

# 运行
ubuntu@VM-1-15-ubuntu:~$ sudo docker run federated-download:v0.0.1
2022-11-03 05:56:57.531568: W tensorflow/stream_executor/platform/default/dso_loader.cc:55] Could not load dynamic library 'libcuda.so.1'; dlerror: libcuda.so.1: cannot open shared object file: No such file or directory
2022-11-03 05:56:57.531673: E tensorflow/stream_executor/cuda/cuda_driver.cc:313] failed call to cuInit: UNKNOWN ERROR (303)
2022-11-03 05:56:57.531708: I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:156] kernel driver does not appear to be running on this host (ac30a6317b98): /proc/driver/nvidia/version does not exist
2022-11-03 05:56:57.532244: I tensorflow/core/platform/cpu_feature_guard.cc:143] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
2022-11-03 05:56:57.559485: I tensorflow/core/platform/profile_utils/cpu_utils.cc:102] CPU Frequency: 2494085000 Hz
2022-11-03 05:56:57.559964: I tensorflow/compiler/xla/service/service.cc:168] XLA service 0x7fb718000b60 initialized for platform Host (this does not guarantee that XLA will be used). Devices:
2022-11-03 05:56:57.560008: I tensorflow/compiler/xla/service/service.cc:176]   StreamExecutor device (0): Host, Default Version
Downloading data from https://storage.googleapis.com/tensorflow/tf-keras-datasets/mnist.npz
11493376/11490434 [==============================] - 1s 0us/step
ubuntu@VM-1-15-ubuntu:~$
```

没法持续运行。
## 5.4 创建deployment部署上k8s测试

```bash
# 创建deployment的yaml文件
ubuntu@VM-1-15-ubuntu:~$ vim federated-dp.yaml
# 创建service的yaml文件
ubuntu@VM-1-15-ubuntu:~$ vim federated-svc.yaml

ubuntu@VM-1-15-ubuntu:~$ ls
federated-dp.yaml  federated-svc.yaml
# 部署deployment
ubuntu@VM-1-15-ubuntu:~$ kubectl apply -f federated-dp.yaml 
deployment.apps/federated-deployment created
# 查看，暂时正常
ubuntu@VM-1-15-ubuntu:~$ kubectl get all
NAME                                        READY   STATUS    RESTARTS   AGE
pod/federated-deployment-54f5db67fd-g8p9j   1/1     Running   0          6s
pod/federated-deployment-54f5db67fd-tdwdl   1/1     Running   0          6s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/federated-deployment   2/2     2            2           6s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/federated-deployment-54f5db67fd   2         2         2       6s
# 部署service
ubuntu@VM-1-15-ubuntu:~$ kubectl apply -f federated-svc.yaml 
service/federated-service created

# 查看，暂时也正常
ubuntu@VM-1-15-ubuntu:~$ kubectl get all
NAME                                        READY   STATUS    RESTARTS   AGE
pod/federated-deployment-54f5db67fd-g8p9j   1/1     Running   0          35s
pod/federated-deployment-54f5db67fd-tdwdl   1/1     Running   0          35s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/federated-service   NodePort    172.16.253.172   <none>        80:30000/TCP   3s
service/kubernetes          ClusterIP   172.16.252.1     <none>        443/TCP        37m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/federated-deployment   2/2     2            2           36s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/federated-deployment-54f5db67fd   2         2         2       36s
# 过一会儿查看，出错
ubuntu@VM-1-15-ubuntu:~$ kubectl get all
NAME                                        READY   STATUS             RESTARTS   AGE
pod/federated-deployment-54f5db67fd-2hn6s   0/1     Evicted            0          33s
pod/federated-deployment-54f5db67fd-ddxts   0/1     Evicted            0          33s
pod/federated-deployment-54f5db67fd-dn9dn   0/1     Evicted            0          33s
pod/federated-deployment-54f5db67fd-g8p9j   0/1     Evicted            0          113s
pod/federated-deployment-54f5db67fd-jkrfn   0/1     Evicted            0          33s
pod/federated-deployment-54f5db67fd-jwfqp   0/1     Evicted            0          33s
pod/federated-deployment-54f5db67fd-mjxk8   0/1     Evicted            0          33s
pod/federated-deployment-54f5db67fd-mm8b6   0/1     Evicted            0          32s
pod/federated-deployment-54f5db67fd-tdwdl   1/1     Running            0          113s
pod/federated-deployment-54f5db67fd-wh6qk   0/1     ImagePullBackOff   0          32s
pod/federated-deployment-54f5db67fd-xdktv   0/1     Evicted            0          33s

NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/federated-service   NodePort    172.16.253.172   <none>        80:30000/TCP   80s
service/kubernetes          ClusterIP   172.16.252.1     <none>        443/TCP        39m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/federated-deployment   1/2     2            1           113s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/federated-deployment-54f5db67fd   2         2         1       113s
```

失败
# 6 搭建kubesphere
## 6.1 安装

```bash
1 安装
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.3.0/kubesphere-installer.yaml
2 下载配置文件
wget https://github.com/kubesphere/ks-installer/releases/download/v3.3.0/cluster-configuration.yaml
3 更改配置文件
vim cluster-configuration.yaml 
4 部署配置文件
kubectl apply -f cluster-configuration.yaml
5 查看部署过程
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l 'app in (ks-install, ks-installer)' -o jsonpath='{.items[0].metadata.name}') -f
```

## 6.2 查看
![在这里插入图片描述](https://img-blog.csdnimg.cn/195a4e612c82454abc675e5cc91dbbef.png)

安装完成，虽然报错但可以正常登录（是因为服务器资源不足的原因，过一会会变好）
## 6.3 登录
![在这里插入图片描述](https://img-blog.csdnimg.cn/791671e9152c493eaafe984853e78e12.png)

# 7 用kubesphere部署
## 7.1 关联coding制品仓库
1. 获取coding制品仓库的配置令牌
2. kubesphere中创建秘钥
 配置-保密字典-创建
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/f1b5f7a4629b44e2af8bf344dcebd25e.png)

## 7.2 创建工作负载-部署（deployment）
应用负载-工作负载-创建
![在这里插入图片描述](https://img-blog.csdnimg.cn/6b37cceae77841178cfc4c1424499e2b.png)

成功！
# 8 创建服务来访问应用（NodePort方式）
## 8.1 kubesphere创建service
kubesphere-应用负载-服务-创建
![在这里插入图片描述](https://img-blog.csdnimg.cn/5b55145a85664707bf17ee1f7a9864e4.png)

## 8.2 访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/caa1d89a293b4645b31351357119cdab.png)

失败！
# 9 更改源码
## 9.1 更改源码依赖的版本
python改为3.9版本
![在这里插入图片描述](https://img-blog.csdnimg.cn/91b64a2e3c0a4a29a5dce5e459419638.png)

## 9.2 构建计划制作镜像推送到制品仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/2a2f074764de453185d1c0b648b96247.png)

# 10 kubesphere重新创建dp和svc
![在这里插入图片描述](https://img-blog.csdnimg.cn/0f2ea632530843bc8c446646b1f8f972.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d2113b566be44b2fada3fbaa53eaa47f.png)

# 11 访问
![在这里插入图片描述](https://img-blog.csdnimg.cn/a30b56dbeb9b4d5bac98e81518c5c36f.png)

可以访问，并且可以正常下载。

至此，实践成功！

`源码开源，若对您有所帮助，请不要吝啬您的star，开放交流学习才能进步！`
