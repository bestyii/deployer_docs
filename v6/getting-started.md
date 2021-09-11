# 新手入门

首先, 我们来 [安装 Deployer](installation.md). 在终端运行以下命令:

```sh
curl -LO https://deployer.org/deployer.phar
mv deployer.phar /usr/local/bin/dep
chmod +x /usr/local/bin/dep
```

现在你就可以通过命令 `dep` 来使用Deployer. 
打开终端，在你项目的目录中运行初始化命令:

```sh
dep init
```

这个命令会在当前目录创建 `deploy.php` 文件. 他被称之为 *食谱（recipe）* 包含了部署相关的配置与任务.
默认情况下，所有的食谱都继承自 [common](https://github.com/deployphp/deployer/blob/master/recipe/common.php) . 把你的 _deploy.php_ 放在项目的根目录中，输入命令 `dep` 或 `dep list` . 将会看到全部可用的命令集.

> 你也可以在项目中的任何子目录中调用 `dep` 命令.

## 创建任务

定义你的个性化任务非常简单:
 
```php
task('test', function () {
    writeln('Hello phpdeployer.com');
});
```

执行这个任务, 运行命令:

```sh
dep test
```

接下来会输出:

```text
➤ Executing task test
Hello phpdeployer.com
✔ Ok
```
## 在远端执行任务
我们要想在远端执行任务，必须要先配置 deployer. 
新创建的 `deploy.php` 文件, 应该包含 `host` 声明, 如:
 
```php
host('phpdeployer.com')
    ->stage('production')    
    ->set('deploy_path', '/var/www/phpdeployer_com');
```

> 也可以在单独的yaml文件中声明主机. 了解更多请参考: [inventory](hosts.md#inventory-file).

更多配置参考:[主机](hosts.md). 

接下来我们定义一个任务,这个任务会在远程主机中执行linux `pwd` 命令:
 
```php
task('pwd', function () {
    $result = run('pwd');
    writeln("当前目录: $result");
});
```

运行命令 `dep pwd`, 你将得到如下结果:

```text
➤ Executing task pwd
当前目录: /var/www/phpdeployer_com
✔ Ok
```

好啦，来准备我们的第一个部署工作. 你需要配置一些参数,如 `repository`, `shared_files,` 以及其它:
   
```php
set('repository', 'git@domain.com:username/repository.git');
set('shared_files', [...]);
```

你可以在任何一个任务中通过`get`方法调用这个参数.
还可以在任何一个主机的声明中覆盖这个参数:
```php
host('phpdeployer.com')
    ...
    ->set('shared_files', [...]);
```

更多部署配置参考: [配置](configuration.md) .


现在来部署我们的程序:
 
```sh
dep deploy
```

要在输出中包含额外的详细信息，可以使用 `--verbose` 选项增加详细程度:
* `-v`   标准输出
* `-vv`  详细输出
* `-vvv` debug
 
Deployer 将在主机上创建以下目录:

* `releases`  包含各版本的文件目录
* `shared` 包含共享的文件和目录
* `current` 软链接到当前版本

将主机的公用目录配置为 `current` 向外提供服务

> ⚠️ 注意: that deployer 默认情况下，采用 [ACL](https://en.wikipedia.org/wiki/Access_control_list) 设置权限.
> 可以使用 `writable_mode` 配置更改此行为. 

默认情况下 deployer 保留5个版本, 你可以通过下面参数增量:
 
```php
set('keep_releases', 10);
```

如果部署过程中出现错误，或者新版本有问题，只需运行以下命令即可回滚到上一个可运行的版本:

```sh
dep rollback
```

您可能希望在其他任务之前(之后)运行某些任务。这真的很简单！

在 `deploy` 任务完成之后重载php-fpm:
```php
task('reload:php-fpm', function () {
    run('sudo service php-fpm reload');
});

after('deploy', 'reload:php-fpm');
```

如果您需要连接到主机，Deployer提供了一个快捷方式:

~~~sh
dep ssh
~~~

此命令将连接到选定的主机,并且进入到 `current_path`.

了解更多部署配置请参考: [配置](configuration.md) . 
