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

## 2.1 设置 setting 添加环境变量
action -> new repository secret
![image](https://github.com/user-attachments/assets/e107b0c5-565a-4d9d-b2d1-fb645978efef)
![image](https://github.com/user-attachments/assets/4d463aee-16d9-49e4-ab42-b71a3a392be3)
![image](https://github.com/user-attachments/assets/7f63bc06-3d5d-47ed-ac48-e3dca4368607)



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

### 1）设置阿里云 Docker Registry 密码访问凭证：(ALIYUN_REGISTRY_PASSWORD)
` ****

### 2） 创建命名空间: ALIYUN_NAME_SPACE 
` your-image-namespace

###  3）阿里云 Docker Registry 用户名: ALIYUN_REGISTRY_USER
` ********

### 4）阿里云地址  ALIYUN_REGISTRY
` registry.cn-hangzhou.aliyuncs.com


## 2.2 创建 GitHub Actions 工作流文件：
### 2.2.1 在仓库中创建一个配置文件，例如 config.env，其中包含仓库名称和标签等信息。例如：
```# 比方mysql
REPO_NAME=mysql
IMAGE_TAG=latest
```
### 2.2.2 查找镜像：
在docerk网站查找你想要下载的镜像，注意对应的版本tag
https://hub.docker.com/
![image](https://github.com/user-attachments/assets/ea2e610e-10f1-4f9d-a379-567992004ba9)

![image](https://github.com/user-attachments/assets/8fb15b95-5716-4ed3-b07d-d53162bd3bb2)


### 2.3 创建 GitHub Actions 工作流文件：
新建 GitHub 仓库，创建一个工作流文件（.github/workflows/ 目录下）。创建一个名为 docker-publish.yml 的文件。

编写 GitHub Actions 工作流：
在 docker-publish.yml 文件中，编写以下工作流内容：

```
# 名称
name: Push Docker Image to Alibaba Cloud

on:
 # 出发的动作
  push:
    branches:
      - main  # 触发工作流的分支

env:
  ALICLOUD_REGISTRY: "${{ secrets.ALICLOUD_REGISTRY }}"
  ALICLOUD_NAME_SPACE: "${{ secrets.ALICLOUD_NAME_SPACE }}"
  ALICLOUD_USERNAME: "${{ secrets.ALICLOUD_USERNAME }}"
  ALICLOUD_PASSWORD: "${{ secrets.ALICLOUD_PASSWORD }}"
  
jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Log in to Alibaba Cloud Docker Registry
        run: |
          echo $ALICLOUD_PASSWORD | docker login --username=$ALICLOUD_USERNAME $ALICLOUD_REGISTRY --password-stdin
      - name: Load configuration
        id: load-config
        run: |
          source config.env
          echo "REPO_NAME=$REPO_NAME" >> $GITHUB_ENV
          echo "IMAGE_TAG=$IMAGE_TAG" >> $GITHUB_ENV
        shell: bash

      - name: Pull Docker image from Docker Hub
        run: |
          docker pull ${REPO_NAME}:${IMAGE_TAG}

      - name: Verify Docker image
        run: |
          docker images ${REPO_NAME}:${IMAGE_TAG}

      - name: Tag Docker image for Alibaba Cloud
        run: |
          docker tag ${REPO_NAME}:${IMAGE_TAG} $ALICLOUD_REGISTRY/$ALICLOUD_NAME_SPACE/${REPO_NAME}:${IMAGE_TAG}

      - name: Push Docker image to Alibaba Cloud
        run: |
          docker push $ALICLOUD_REGISTRY/$ALICLOUD_NAME_SPACE/${REPO_NAME}:${IMAGE_TAG}

      - name: Logout from Docker registry
        run: docker logout $ALIYUN_REGISTRY

```
**解释**
- Checkout repository：使用 actions/checkout 动作将你的代码库签出到 runner 上。
- Load configuration：加载 config.env 文件中的配置，并将变量添加到 GitHub Actions 的环境变量中。
- Log in to Alibaba Cloud Docker Registry：使用阿里云的凭据登录到 Docker Registry。
- Pull Docker image from Docker Hub：从 Docker Hub 拉取指定的镜像。
- Verify Docker image：验证镜像是否正确拉取，并输出镜像列表。
- Tag Docker image for Alibaba Cloud：为拉取的镜像打上阿里云私有仓库的标签。
- Push Docker image to Alibaba Cloud：将打好标签的镜像推送到阿里云私有仓库。
- Logout from Docker registry：从 Docker Registry 注销。



### 2.4 配置 GitHub 仓库的 Secrets：
在 GitHub 仓库中，配置以下 Secrets，以便工作流能够访问阿里云凭据：

- ALICLOUD_USERNAME: 阿里云 Docker Registry 用户名。
- ALICLOUD_PASSWORD: 阿里云 Docker Registry 密码。
- ALICLOUD_REGISTRY: 阿里云地址
- ALICLOUD_NAME_SPACE：命名空间


### 添加secrets
在 GitHub 仓库的 Settings -> Secrets and variables -> Actions 下添加这些 Secrets。

### 触发 GitHub Actions 工作流：
每当向指定的分支（例如 main 分支）推送代码时，GitHub Actions 工作流将会自动运行，构建 Docker 镜像并推送到阿里云私有仓库。

## 开始工作

![image](https://github.com/user-attachments/assets/20a660af-1572-4066-b3d2-37b58fe880bc)


校验是否运行成功

![image](https://github.com/user-attachments/assets/d491da29-4082-450a-93e3-a6e74accee49)



