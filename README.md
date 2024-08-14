构建多平台支持的 Docker 镜像是为了确保你的应用可以在不同架构（如 x86_64、ARM）和操作系统（如 Linux、Windows）上运行。

可以通过 Docker 提供的 `buildx` 工具来实现多平台镜像构建。以下是详细步骤：

## 1. 安装和配置 `buildx`

`buildx` 是 Docker 提供的一个扩展工具，允许你构建支持多平台的镜像。

- **确保 Docker 版本最新**：`buildx` 是 Docker 19.03 版本及以上支持的。

- **启用 `buildx`**：
  ```bash
  docker buildx create --use
  ```

- **查看当前的 `builder` 实例**：
  ```bash
  docker buildx ls
  ```

### 2. 构建多平台镜像
使用 `docker buildx build` 命令构建支持多平台的 Docker 镜像。

- **创建一个简单的 Dockerfile**：
  ```dockerfile
  FROM php:8.2.22-alpine

  # Add docker-php-extension-installer script
  ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

  # Install dependencies
  RUN apk add --no-cache \
      bash \
      curl \
      freetype-dev \
      g++ \
      gcc \
      git \
      icu-dev \
      icu-libs \
      libc-dev \
      libzip-dev \
      make \
      mysql-client \
      nodejs \
      npm \
      oniguruma-dev \
      yarn \
      openssh-client \
      postgresql-libs \
      rsync \
      zlib-dev

  # Install php extensions
  RUN chmod +x /usr/local/bin/install-php-extensions && \
      install-php-extensions \
      @composer \
      redis-stable \
      imagick-stable \
      xdebug-stable \
      bcmath \
      calendar \
      exif \
      gd \
      intl \
      pdo_mysql \
      pdo_pgsql \
      pcntl \
      soap \
      zip

  # Add local and global vendor bin to PATH.
  ENV PATH ./vendor/bin:/composer/vendor/bin:/root/.composer/vendor/bin:/usr/local/bin:$PATH

  # Install PHP_CodeSniffer
  RUN composer global require "squizlabs/php_codesniffer=*" "laravel/pint=*"

  # Setup working directory
  WORKDIR /var/www
  ```

- **构建并推送多平台镜像**：
  使用以下命令来构建多平台支持的 Docker 镜像：
  ```bash
  docker buildx build --platform linux/amd64,linux/arm64 -t curder/php-docker --push .
  ```

    - `--platform`：指定目标平台，例如 `linux/amd64`（x86_64 架构）和 `linux/arm64`（ARM 架构）。
    - `-t`：为镜像指定标签。
    - `--push`：将构建的镜像推送到 Docker 注册表（如 Docker Hub）。

### 3. 验证多平台镜像
在推送镜像后，你可以通过以下命令来验证它支持的平台：

```bash
docker run --rm --platform linux/arm64 curder/php-docker
docker run --rm --platform linux/amd64 curder/php-docker
```

### 4. 额外提示
- **QEMU 支持**：为了在 x86_64 主机上构建 ARM 架构的镜像，Docker 会使用 QEMU 进行跨架构仿真。如果你需要支持更多架构，可能需要安装和配置 QEMU。

  ```bash
  docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
  ```

- **检查镜像支持的架构**：
  ```bash
  docker manifest inspect curder/php-docker
  ```

这样你就可以创建一个能够在多平台上运行的 Docker 镜像了。