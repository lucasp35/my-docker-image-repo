# my-docker-image-repo
my-docker-image-repo docker镜像



# 2 配置github 
使用 GitHub Actions 将 Docker 镜像推送到自己的阿里云私有仓库

## 2.1  配置文件
在仓库中创建一个配置文件，例如 config.env，其中包含仓库名称和标签等信息。例如：
```
# 比方mysql
REPO_NAME=mysql
IMAGE_TAG=latest
```

## 2.2 创建 GitHub Actions 工作流文件：
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
  IMAGE: "${REPO_NAME}:${IMAGE_TAG}"
  
jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Log in to Alibaba Cloud Docker Registry
        run: |
          echo $ALICLOUD_PASSWORD | docker login --username=$ALICLOUD_USERNAME $ALICLOUD_REGISTRY --password-stdin

      - name: Build Docker image
        run: |
          docker build -t $ALICLOUD_REGISTRY/$ALICLOUD_NAME_SPACE/$IMAGE .

      - name: Push Docker image to Alibaba Cloud
        run: |
          docker push $ALICLOUD_REGISTRY/$ALICLOUD_NAME_SPACE/$IMAGE

      - name: Logout from Docker registry
        run: docker logout $ALIYUN_REGISTRY

```


## 2.3 配置 GitHub 仓库的 Secrets：
在 GitHub 仓库中，配置以下 Secrets，以便工作流能够访问阿里云凭据：

- ALICLOUD_USERNAME: 阿里云 Docker Registry 用户名。
- ALICLOUD_PASSWORD: 阿里云 Docker Registry 密码。
- ALICLOUD_REGISTRY: 阿里云地址
- ALICLOUD_NAME_SPACE：命名空间


### 添加secrets
在 GitHub 仓库的 Settings -> Secrets and variables -> Actions 下添加这些 Secrets。

### 触发 GitHub Actions 工作流：
每当向指定的分支（例如 main 分支）推送代码时，GitHub Actions 工作流将会自动运行，构建 Docker 镜像并推送到阿里云私有仓库。
