## 使用 GitHub Actions 将 Docker 镜像推送到自己的阿里云私有仓库，解决一些国外镜像无法拉取，并且国内的镜像源网站并未更新该镜像的情况 
## 工作原理：借助GitHub Actions 自动从国外Docker镜像网站（hub.docker.com）拉取镜像并push到自己的阿里云私有仓库。 
> 由于项目在GitHub上，本身就在国外，访问Docker网站无障碍，拉取对应的镜像后添加至自己的阿里云仓库，再从阿里云私有仓库下载，解决了网络不通的问题。

### 关于GitHub Actions
>GitHub Actions 是 GitHub 提供的一种 CI/CD（持续集成和持续部署）服务，允许你自动化软件开发工作流。你可以创建工作流来构建、测试和部署代码，或者执行其他自动化任务，例如发布 GitHub Releases、发送通知等。
### GitHub Actions 重要组成
- 1.工作流（Workflow）
定义自动化过程，由一个或多个作业（Job）组成。工作流文件是用 YAML 编写的，存储在仓库的 .github/workflows/ 目录中。
- 2.作业（Job）
工作流中的一个任务或一组任务，包含一个或多个步骤（Step）。作业在虚拟环境中运行（由Github提供），可以指定运行在不同的平台上（如 Ubuntu、Windows、macOS）。
- 3.步骤（Step）
作业中的单个任务，可以是运行命令或调用动作（Action）。步骤按顺序执行。
- 4.动作（Action）
可重用的命令集合，可以在工作流中调用。GitHub 提供了许多预定义的动作，你也可以创建自定义动作。

准备工作
- [注册阿里云账号，创建个人实例，设置命名空间等](#chapter1)
- [ 配置github，配置自动化任务，push镜像至私有仓库](#chapter2)
- [拉取自己构建的镜像](#chapter3)
- [查看镜像文件](#chapter4) 



<a id = "chapter1"/>
# 1. 阿里云镜像服务地址：
> https://cr.console.aliyun.com/

注册自己的账号，进入容器镜像服务

![image](https://github.com/user-attachments/assets/571c1b9f-b5c3-4538-912a-491ad5558626)

- 创建命名空间
- 查看访问凭证

![image](https://github.com/user-attachments/assets/06b410df-2f8e-4d9b-855f-9946f4c26cb7)


> username是用户名
> registry.cn-hangzhou.aliyuncs.com 是阿里云镜像服务地址


![image](https://github.com/user-attachments/assets/2ad50f6a-65ff-4448-8674-2f116569e864)


<a id = "chapter2"/>
# 2 配置github 
> 使用 GitHub Actions 将 Docker 镜像推送到自己的阿里云私有仓库
## Fork本项目
> fork一份到自己的仓库中


## 2.1  配置 GitHub 仓库的 Secrets：
> 在 GitHub 仓库的 **Settings** -> **Secrets and variables** -> Actions 下添加这些 Secrets, 以便工作流能够访问阿里云凭据：

### 2.1.1 配置 GitHub 仓库的 Secrets：
- ALICLOUD_USERNAME: 阿里云 Docker Registry 用户名。
- ALICLOUD_PASSWORD: 阿里云 Docker Registry 密码。
- ALICLOUD_REGISTRY: 阿里云地址 registry.cn-hangzhou.aliyuncs.com
- ALICLOUD_NAME_SPACE：命名空间
  
![image](https://github.com/user-attachments/assets/e107b0c5-565a-4d9d-b2d1-fb645978efef)
![image](https://github.com/user-attachments/assets/4d463aee-16d9-49e4-ab42-b71a3a392be3)
![image](https://github.com/user-attachments/assets/7f63bc06-3d5d-47ed-ac48-e3dca4368607)



## 2.2  配置文件
> 在仓库中创建一个配置文件，例如 config.env，其中包含仓库名称和标签等信息。例如：

```powershell
# 比方mysql
REPO_NAME=mysql
IMAGE_TAG=latest
```
![image](https://github.com/user-attachments/assets/78277fef-f53d-429a-8fda-12b42e5f0aa9)

在该config.env中添加自己想添加的镜像文件
![image](https://github.com/user-attachments/assets/31b0b9a1-28db-445c-b680-49f609480375)


## 2.3 创建 GitHub Actions 工作流文件：
### 2.3.1 在仓库中创建一个配置文件，例如 config.env，其中包含仓库名称和标签等信息。例如：
```xml # 比方mysql
REPO_NAME=mysql
IMAGE_TAG=latest
```
### 2.3.2 查找镜像：
在docerk网站查找你想要下载的镜像，注意对应的版本tag
> https://hub.docker.com/

![image](https://github.com/user-attachments/assets/ea2e610e-10f1-4f9d-a379-567992004ba9)

![image](https://github.com/user-attachments/assets/8fb15b95-5716-4ed3-b07d-d53162bd3bb2)


## 2.4 启动Action
> 进入到fork的项目，点击Action按钮，启用Github Action功能

## 开始工作

![image](https://github.com/user-attachments/assets/20a660af-1572-4066-b3d2-37b58fe880bc)


> 校验是否运行成功


![image](https://github.com/user-attachments/assets/d491da29-4082-450a-93e3-a6e74accee49)


## 触发 GitHub Actions 工作流：
> 每当向指定的分支（例如 main 分支）推送代码时（或者修改config.env 文件），GitHub Actions 工作流将会自动运行，构建 Docker 镜像并推送到阿里云私有仓库。



<a id = "chapter3"/>
# 3 拉取自己构建的镜像：
![image](https://github.com/user-attachments/assets/913fa745-18fd-45b7-af27-badfbc42f920)

管理镜像，里面告诉了如何拉取镜像，如果命名空间是私有的，需要先登录
```
$ docker login --username=username registry.cn-hangzhou.aliyuncs.com
```
再拉取镜像
```
$ docker pull registry.cn-hangzhou.aliyuncs.com/namespace/mysql:[镜像版本号]
```

![image](https://github.com/user-attachments/assets/ec4745df-b48f-43e5-91c6-8b1d7970a39a)



**注意** 
命名空间设置成公开后，拉取自己的仓库镜像不需要登录，否则，每次拉取镜像前需要登录验证，两种方式均可
![image](https://github.com/user-attachments/assets/f4f6f206-be32-4d36-875c-7f20e051380b)


<a id = "chapter4"/>
# 4 查看镜像文件
进入自己的阿里云个人仓库 -> 个人实例 - 仓库管理 -> 镜像仓库
![image](https://github.com/user-attachments/assets/553c4379-2bfa-44f6-9d09-c1c2d1f07370)

进入对应的容器
````
从Registry中拉取镜像
$ docker pull registry.cn-hangzhou.aliyuncs.com/docker-image-pull/mysql:latest
````




