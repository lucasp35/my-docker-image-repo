## 使用 GitHub Actions 将 Docker 镜像推送到自己的阿里云私有仓库，解决一些国外镜像无法拉取，并且国内的镜像源网站并未更新该镜像的情况 


# 1. 阿里云镜像服务地址：
` https://cr.console.aliyun.com/
注册自己的账号，进入容器镜像服务
![image](https://github.com/user-attachments/assets/571c1b9f-b5c3-4538-912a-491ad5558626)

- 创建命名空间
- 查看访问凭证

![image](https://github.com/user-attachments/assets/06b410df-2f8e-4d9b-855f-9946f4c26cb7)

username是用户名
registry.cn-hangzhou.aliyuncs.com 是阿里云镜像服务地址
![image](https://github.com/user-attachments/assets/2ad50f6a-65ff-4448-8674-2f116569e864)


# 2 配置github 
使用 GitHub Actions 将 Docker 镜像推送到自己的阿里云私有仓库
## Fork本项目
fork一份到自己的仓库中


## 2.1 设置 setting 添加环境变量
action -> new repository secret
![image](https://github.com/user-attachments/assets/e107b0c5-565a-4d9d-b2d1-fb645978efef)
![image](https://github.com/user-attachments/assets/4d463aee-16d9-49e4-ab42-b71a3a392be3)
![image](https://github.com/user-attachments/assets/7f63bc06-3d5d-47ed-ac48-e3dca4368607)

### 2.1.1 配置 GitHub 仓库的 Secrets：
在 GitHub 仓库中，配置以下 Secrets，以便工作流能够访问阿里云凭据：

- ALICLOUD_USERNAME: 阿里云 Docker Registry 用户名。
- ALICLOUD_PASSWORD: 阿里云 Docker Registry 密码。
- ALICLOUD_REGISTRY: 阿里云地址 registry.cn-hangzhou.aliyuncs.com
- ALICLOUD_NAME_SPACE：命名空间


### 2.2.2 添加secrets
在 GitHub 仓库的 Settings -> Secrets and variables -> Actions 下添加这些 Secrets。



## 2.2  配置文件
在仓库中创建一个配置文件，例如 config.env，其中包含仓库名称和标签等信息。例如：
```
# 比方mysql
REPO_NAME=mysql
IMAGE_TAG=latest
```
![image](https://github.com/user-attachments/assets/78277fef-f53d-429a-8fda-12b42e5f0aa9)

在该config.env中添加自己想添加的镜像文件
![image](https://github.com/user-attachments/assets/31b0b9a1-28db-445c-b680-49f609480375)


## 2.3 创建 GitHub Actions 工作流文件：
### 2.3.1 在仓库中创建一个配置文件，例如 config.env，其中包含仓库名称和标签等信息。例如：
```# 比方mysql
REPO_NAME=mysql
IMAGE_TAG=latest
```
### 2.3.2 查找镜像：
在docerk网站查找你想要下载的镜像，注意对应的版本tag
https://hub.docker.com/
![image](https://github.com/user-attachments/assets/ea2e610e-10f1-4f9d-a379-567992004ba9)

![image](https://github.com/user-attachments/assets/8fb15b95-5716-4ed3-b07d-d53162bd3bb2)


## 2.4 启动Action
进入到fork的项目，点击Action按钮，启用Github Action功能

## 开始工作

![image](https://github.com/user-attachments/assets/20a660af-1572-4066-b3d2-37b58fe880bc)


校验是否运行成功

![image](https://github.com/user-attachments/assets/d491da29-4082-450a-93e3-a6e74accee49)


## 触发 GitHub Actions 工作流：
每当向指定的分支（例如 main 分支）推送代码时（或者修改config.env 文件），GitHub Actions 工作流将会自动运行，构建 Docker 镜像并推送到阿里云私有仓库。



