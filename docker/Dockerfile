# 使用官方Python镜像作为基础镜像
FROM swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/python:3.12-slim

# 设置时区环境变量
ENV TZ=Asia/Shanghai

# 替换apt-get源为国内源（阿里源）
RUN echo "deb https://mirrors.aliyun.com/debian/ bookworm main" > /etc/apt/sources.list && \
    echo "deb https://mirrors.aliyun.com/debian/ bookworm-updates main" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.aliyun.com/debian-security bookworm-security main" >> /etc/apt/sources.list && \
    rm -rf /etc/apt/sources.list.d/* || true

# 设置工作目录
WORKDIR /app

# 安装基础依赖、中文字体和时区数据
RUN apt-get update && apt-get install -y \
    curl \
    tar \
    xz-utils \
    ca-certificates \
    fonts-noto-cjk \
    fonts-wqy-zenhei \
    fonts-wqy-microhei \
    fontconfig \
    tzdata \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone \
    && fc-cache -fv \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 安装Node.js（从淘宝镜像下载二进制包，最稳定快速的方案）
RUN NODE_VERSION=18.20.4 && \
    ARCH=$(dpkg --print-architecture) && \
    if [ "$ARCH" = "amd64" ]; then NODE_ARCH="x64"; \
    elif [ "$ARCH" = "arm64" ]; then NODE_ARCH="arm64"; \
    else NODE_ARCH="$ARCH"; fi && \
    curl -fsSL https://registry.npmmirror.com/-/binary/node/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-${NODE_ARCH}.tar.gz -o /tmp/node.tar.gz && \
    tar -xzf /tmp/node.tar.gz -C /usr/local --strip-components=1 && \
    rm /tmp/node.tar.gz && \
    ln -s /usr/local/bin/node /usr/local/bin/nodejs

# 验证安装
RUN node --version && npm --version

# 配置npm使用淘宝镜像源
RUN npm config set registry https://registry.npmmirror.com/

# 安装uv
RUN pip install uv

# 复制项目文件
COPY pyproject.toml .
COPY src/ ./src/
COPY app.py run.py ./

# 安装Python依赖
RUN uv sync --no-dev

# 复制其余项目文件
COPY . .

# 创建必要的目录
RUN mkdir -p /app/data /app/conf && chmod 777 /app/data /app/conf

# 暴露Streamlit默认端口
EXPOSE 8503

# 设置健康检查
HEALTHCHECK CMD curl --fail http://localhost:8503/_stcore/health || exit 1

# 启动应用
CMD ["uv", "run", "streamlit", "run", "app.py", "--server.port=8503", "--server.address=0.0.0.0"]
