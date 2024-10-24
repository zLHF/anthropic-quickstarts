# Anthropic 计算机使用演示

> [!注意]
> 计算机使用是一个测试版功能。请注意，计算机使用功能带来的风险与标准 API 功能或聊天界面的风险有所不同。在使用计算机功能与互联网交互时，这些风险会更加显著。为了最小化风险，请考虑采取以下预防措施：
>
> 1. 使用具有最小权限的专用虚拟机或容器，以防止直接系统攻击或意外。
> 2. 避免让模型接触敏感数据（如账户登录信息），以防信息泄露。
> 3. 将互联网访问限制在允许名单内的域名范围内，以减少接触恶意内容的风险。
> 4. 对于可能产生实际影响的决策，以及任何需要明确同意的任务（如接受 cookie、执行金融交易或同意服务条款），都应请求人工确认。
>
> 在某些情况下，Claude 会执行在内容中发现的命令，即使这些命令与用户指令相冲突。例如，网页上的指令或图片中包含的指令可能会覆盖用户指令或导致 Claude 出错。我们建议采取预防措施，将 Claude 与敏感数据和操作隔离，以避免与提示注入相关的风险。
>
> 最后，在您自己的产品中启用计算机使用功能之前，请告知最终用户相关风险并获得他们的同意。

本代码仓库帮助您开始使用 Claude 的计算机功能，包含以下参考实现：

* 用于创建包含所有必要依赖项的 Docker 容器的构建文件
* 使用 Anthropic API、Bedrock 或 Vertex 访问更新后的 Claude 3.5 Sonnet 模型的计算机使用代理循环
* Anthropic 定义的计算机使用工具
* 用于与代理循环交互的 streamlit 应用

请使用[此表单](https://forms.gle/BT1hpBrqDPDUrCqo7)提供关于模型响应质量、API 本身或文档质量的反馈 - 我们期待听到您的意见！

> [!重要]
> 本参考实现中使用的测试版 API 可能会发生变化。请参考 [API 发布说明](https://docs.anthropic.com/en/release-notes/api)获取最新信息。

> [!重要]
> 各组件之间是弱分离的：代理循环在被 Claude 控制的容器中运行，每次只能被一个会话使用，如有必要必须在会话之间重启或重置。

## 快速入门：运行 Docker 容器

### Anthropic API

> [!提示]
> 您可以在 [Anthropic 控制台](https://console.anthropic.com/)中找到您的 API 密钥。

```bash
export ANTHROPIC_API_KEY=%your_api_key%
docker run \
    -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

容器运行后，请参考下面的[访问演示应用](#访问演示应用)部分了解如何连接到界面。

### Bedrock

您需要传入具有适当权限的 AWS 凭证才能在 Bedrock 上使用 Claude。

您有几种方式可以与 Bedrock 进行身份验证。更多详情和选项请参见 [boto3 文档](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html#environment-variables)。

#### 选项 1：（建议）使用主机的 AWS 凭证文件和 AWS 配置文件

```bash
export AWS_PROFILE=<your_aws_profile>
docker run \
    -e API_PROVIDER=bedrock \
    -e AWS_PROFILE=$AWS_PROFILE \
    -e AWS_REGION=us-west-2 \
    -v $HOME/.aws/credentials:/home/computeruse/.aws/credentials \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

容器运行后，请参考下面的[访问演示应用](#访问演示应用)部分了解如何连接到界面。

#### 选项 2：使用访问密钥和密钥

```bash
export AWS_ACCESS_KEY_ID=%your_aws_access_key%
export AWS_SECRET_ACCESS_KEY=%your_aws_secret_access_key%
export AWS_SESSION_TOKEN=%your_aws_session_token%
docker run \
    -e API_PROVIDER=bedrock \
    -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID \
    -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY \
    -e AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN \
    -e AWS_REGION=us-west-2 \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

容器运行后，请参考下面的[访问演示应用](#访问演示应用)部分了解如何连接到界面。

### Vertex

您需要传入具有适当权限的 Google Cloud 凭证才能在 Vertex 上使用 Claude。

```bash
docker build . -t computer-use-demo
gcloud auth application-default login
export VERTEX_REGION=%your_vertex_region%
export VERTEX_PROJECT_ID=%your_vertex_project_id%
docker run \
    -e API_PROVIDER=vertex \
    -e CLOUD_ML_REGION=$VERTEX_REGION \
    -e ANTHROPIC_VERTEX_PROJECT_ID=$VERTEX_PROJECT_ID \
    -v $HOME/.config/gcloud/application_default_credentials.json:/home/computeruse/.config/gcloud/application_default_credentials.json \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it computer-use-demo
```

容器运行后，请参考下面的[访问演示应用](#访问演示应用)部分了解如何连接到界面。

此示例展示了如何使用 Google Cloud 应用程序默认凭证与 Vertex 进行身份验证。

您也可以设置 `GOOGLE_APPLICATION_CREDENTIALS` 来使用任意凭证文件，详情请参见 [Google Cloud 身份验证文档](https://cloud.google.com/docs/authentication/application-default-credentials#GAC)。

### 访问演示应用

容器运行后，在浏览器中打开 [http://localhost:8080](http://localhost:8080) 访问包含代理聊天和桌面视图的组合界面。

容器在 `~/.anthropic/` 中存储设置，如 API 密钥和自定义系统提示。挂载此目录可以在容器重启之间保持这些设置。

其他访问方式：

- 仅 Streamlit 界面：[http://localhost:8501](http://localhost:8501)
- 仅桌面视图：[http://localhost:6080/vnc.html](http://localhost:6080/vnc.html)
- 直接 VNC 连接：`vnc://localhost:5900`（适用于 VNC 客户端）

## 屏幕大小

可以使用环境变量 `WIDTH` 和 `HEIGHT` 设置屏幕大小。例如：

```bash
docker run \
    -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -e WIDTH=1920 \
    -e HEIGHT=1080 \
    -it ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

我们不建议发送分辨率高于 [XGA/WXGA](https://en.wikipedia.org/wiki/Display_resolution_standards#XGA) 的截图，以避免与[图像调整大小](https://docs.anthropic.com/en/docs/build-with-claude/vision#evaluate-image-size)相关的问题。
相比依赖 API 中的图像调整大小功能，直接在工具中实现缩放会获得更高的模型准确性和更快的性能。本项目中的 `computer` 工具实现演示了如何将图像和坐标从更高分辨率缩放到建议的分辨率。

## 开发

```bash
./setup.sh  # 配置 venv、安装开发依赖项和安装预提交钩子
docker build . -t computer-use-demo:local  # 手动构建 docker 镜像（可选）
export ANTHROPIC_API_KEY=%your_api_key%
docker run \
    -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
    -v $(pwd)/computer_use_demo:/home/computeruse/computer_use_demo/ `# 挂载本地 python 模块用于开发` \
    -v $HOME/.anthropic:/home/computeruse/.anthropic \
    -p 5900:5900 \
    -p 8501:8501 \
    -p 6080:6080 \
    -p 8080:8080 \
    -it computer-use-demo:local  # 也可以使用 ghcr.io/anthropics/anthropic-quickstarts:computer-use-demo-latest
```

上面的 docker run 命令将代码仓库挂载到 docker 镜像内部，这样您就可以从主机编辑文件。Streamlit 已经配置了自动重载功能。
