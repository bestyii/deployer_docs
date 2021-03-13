# 安装

有三种方式安装 deployer : 

1. 下载 phar 归档
2. composer 源码安装
3. composer 发布版安装

### 下载 phar 归档

以phar归档方式安装 Deployer , 运行以下命令:

```sh
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

如果你需要Deployer其他版本, 你可以从 [下载](https://deployer.org/download)页面中找到需要的版本.
最后, 更新Deployer到最新版本, 运行以下命令:

```sh
dep self-update
```

要升级到下一个主要版本（如果可用），请使用 `--upgrade (-u)` 选项
```sh
dep self-update --upgrade
```

### composer 源码安装

使用 Composer 源码安装 Deployer，请运行以下命令:

```sh
composer require deployer/deployer --dev
```

您还可以全局安装它:

``` sh
composer global require deployer/deployer
```

更多关于全局安装到介绍请参考: https://getcomposer.org/doc/03-cli.md#global

然后使用Deployer, 请运行以下命令:

```sh
php vendor/bin/dep
```

> 如果你同时使用两种方法安装了 Deployer, 当运行 `dep` 命令时，会优先使用composer安装的版本. 

> 如果存在依赖项冲突，可以使用 "composer 发布版安装"

### composer 发布版安装

使用 Composer 发布版安装 Deployer，请运行以下命令:

```sh
composer require deployer/dist --dev
```

然后使用Deployer, 请运行以下命令:

```sh
php vendor/bin/dep
```

### 构建自己的 phar

如果要从源代码构建Deployer，请从GitHub克隆项目:

```sh
git clone https://github.com/deployphp/deployer.git
```

然后在项目目录中运行以下命令:

```sh
php bin/build
```

将会构建出 `deployer.phar` phar 归档.


### 自动完成

Deployer为bash/zsh/fish提供了一个自动完成脚本，因此您不需要记住所有的任务和选项。
需要安装，请运行以下命令：
~~~bash
dep autocomplete
~~~

并按照说明操作。


接下来请阅读 [新手入门](getting-started.md) .
