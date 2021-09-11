# 配置

若要设置配置参数, 使用 `set` 函数;若要在任务中获取配置参数, 使用 `get` 函数.
```php
set('param', 'value');

task('deploy', function () {
    $param = get('param');
});
```

各个主机可以分别覆盖这些参数:

```php
host(...)
    ->set('param', 'new value');
```

配置参数也可以指定为回调函数, 该函数将在第一次 `get` 调用时在远程主机上执行:

```php
set('current_path', function () {
    return run('pwd');
});
```

可以在调用 `run` 函数中使用带有 `{{ }}` 的参数值, 就像下面这样:

```php
run('cd {{release_path}} && command');
```

替代下面这种写法:

```php
run('cd ' . get('release_path') . ' && command');
```


`common recipe`附带了一些预定义的配置参数, 如下所示. 

获取可用参数的列表, 请运行:

```sh
dep config:dump
```

显示当前部署的版本:

```bash
dep config:current
```

Show inventory:

```bash
dep config:hosts
```



常见变量列表.

### deploy\_path

在远程主机上部署应用程序的位置. 应该为所有主机定义此变量. 
例如, 要将应用程序部署到主目录下:
```php
host(...)
    ->set('deploy_path', '~/project');
```

### hostname

当前主机名. 由 `host` 函数自动设置.

### user

当前用户名. 默认为当前git用户名:

```php
set('user', function () {
    return runLocally('git config --get user.name');
});
```

你可以在 _deploy.php_ 中覆盖它, 如, 使用环境变量中的值:

```php
set('user', function () {
    return getenv('DEP_USER');
});
```

`user` 参数可用于配置通知系统:

```php
set('slack_text', '{{user}} 正在部署分支 {{branch}} 到 {{hostname}}主机上');
```

### release\_path

当前版本目录的完整路径。非部署上下文中的当前目录路径。
将其用作构建的工作路径:

```php
task('build', function () {
    cd('{{release_path}}');
    // ...
});
```

> 默认情况下, 简单任务的工作路径是 `release_path`:
> ```php
> task('build', 'webpack -p');
> ```

### previous\_release

前一个版本的完整路径. (如果第一次发布, 变量不存在)
```php
task('npm', function () {
    if (has('previous_release')) {
        run('cp -R {{previous_release}}/node_modules {{release_path}}/node_modules');
    }
    
    run('cd {{release_path}} && npm install');
});
```

### ssh\_multiplexing

使用 [ OpenSSH 多路复用](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Multiplexing) 提速原生客户端.

```php
set('ssh_multiplexing', true);
```

### default\_stage

默认场景设置. 如果主机有场景的声明，则使用`dep deploy`命令部署时, 自动选择有默认场景声明的主机进行部署。
```php
set('default_stage', 'prod');

host(...)
    ->stage('prod');
```

如果需要复杂的方式来声明场景, 你还可以将可调用的程序(callable)做为参数.

Having callable in set() allows you to not set the value when declaring it, but later when it is used. There is no difference 
when we assign a simple string.
 But when we assign value of a function, then this function must be called at once, if not used as callable. 
With callable, it can be called when used, so a function which determines a variable can be overwritten by 
the user with its own function. This is the great power of having callable in set() instead of direct in function calls.

在`set()`中使用可调用程序时,允许在声明值时不设置该值, 而是在稍后使用该值时设置该值。这种情况与指定一个简单的字符串没有区别。
但当我们给一个函数赋值时，如果这个函数不能作为可调用函数使用，那么它必须立即被调用。
通过callable，可以在使用时调用它，因此声明变量的函数可以被用户用自己的函数覆盖。这是在`set()`中使用callable而不是在函数调用中使用direct的强大功能。
**Example 1: Direct function assign in set()**

Lets assume that we must include some third party recipe that is setting 'default_stage' like this:
```php
set('default_stage', \ThirdPartyVendor\getDefaultStage());
```

And we want to overwrite this in our deploy.php with our own value:
```php
set('default_stage', \MyVendor\getDefaultStage());
```

Third party recipe should avoid a direct function call, because it will be called always even if we overwrite it with 
our own set('default_stage', \MyVendor\getDefaultStage()). Look at the next example how the third party recipe should use
callable in that case.

**Example 2: Callable assign in set()**

Lets assume that we must include some third party recipe that is setting 'default_stage' like this:
```php
set('default_stage', function() {
    return \ThirdPartyVendor\getDefaultStage();
});
```

And we want to overwrite this in our deploy.php:
```php
set('default_stage', function() {
    return \MyVendor\getDefaultStage();
});
```

The result is that only \MyVendor\getDefaultStage() is run.

### keep\_releases

保留的发布版本数量. `-1` 为无限制. 默认值 `5`.

### repository

Git 仓库.

要使用私有库，需要在主机上生成SSH密钥(SSH-key)并将其添加到存储库中作为部署密钥（又称访问密钥）。这个密钥允许你的主机取出代码。或者使用代理转发。

请注意，当主机第一次连接时，它要求在 `known_hosts` 文件中添加主机。
最简单的方法是在主机上运行 `git clone <repo>` 并在提示时说 `yes` 。

### git\_tty

为 `git clone` 命令分配TTY。默认情况下为`false` 。这允许您输入密钥的密码短语或将主机添加到`known_hosts`。

```php
set('git_tty', true);
```

### git\_recursive

为git clone设置 `--recursive` 标志。默认情况下为 `true` 。将此设置为 `false` 将阻止子模块被克隆。

```php
set('git_recursive', false);
```

### branch

要部署的分支.

如果要部署特定的标记或修订，可以在运行 `dep deploy`时使用 `--tag` 和 `--revision` 选项。例如:

```bash
dep deploy --tag="v0.1"
dep deploy --revision="5daefb59edbaa75"
```

请注意 `tag` 的优先级高于 `branch` ，而低于 `revision`。

### shared\_dirs

共享目录列表.

```php
set('shared_dirs', [
    'logs',
    'var',
    ...
]);
```

### shared\_files

共享文件列表.

### copy\_dirs

要在版本之间复制的文件列表.

### writable\_dirs

web服务器中必须可写的目录列表.

### writable\_mode

可写模式

* `acl` (*默认*) 使用 `setfacl` 用于更改目录的ACL.
* `chmod` 使用 unix `chmod` 命令,
* `chown` 使用 unix `chown` 命令,
* `chgrp` 使用 unix `chgrp` 命令,

### writable\_use\_sudo

是否将 `sudo` 与可写命令一起使用。默认为 `false`。

### writable\_chmod\_mode

用于在`writable_mode`的模式时，`chmod`设置的权限 。默认值：`0755`。


### writable\_chmod\_recursive

是否对`chmod` 操作的目录进行递归设置 。默认值：`true`。

### http\_user

运行web服务器的用户。如果未配置此参数，deployer将尝试从进程列表中检测它。

### clear\_paths

更新代码后需要在发布版本中删除的路径列表。

### clear\_use\_sudo

是否使用 `sudo` 在clear\_paths中一起使用。默认为 `false`。

### cleanup\_use\_sudo

是否将 `sudo` 与 `cleanup` 任务一起使用。默认为`false`。

### use\_relative\_symlink

是否使用软连接。默认情况下，deployer将检测系统是否支持软连接并使用它们。
> 如果系统支持，则默认使用软连接.

### use\_atomic\_symlink

是否使用原子符号链接。默认情况下，部署程序将检测系统是否支持原子符号链接并使用它们。
> 如果系统支持，则默认情况下使用原子符号链接.

### composer\_action

Composer 动作. 默认为 `install`.

### composer\_options

Composer 选项.

### env

环境变量数组.

```php
set('env', [
    'VARIABLE' => 'value',
]);
```


了解更多请关注 [任务](tasks.md).
