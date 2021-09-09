# 流水线 (Flow)

如果你的recipe是基于Deployer自带的 *common* recipe 或框架recipe, 则使用的是一个默认的流水线.
每个流水线是在 `deploy` 命名空间内描述的一组其他任务.一个常见的部署流程如下所示:

~~~php
task('deploy', [
    'deploy:prepare',
    'deploy:lock',
    'deploy:release',
    'deploy:update_code',
    'deploy:shared',
    'deploy:writable',
    'deploy:vendors',
    'deploy:clear_paths',
    'deploy:symlink',
    'deploy:unlock',
    'cleanup',
    'success'
]);
~~~

框架recipes可能在流程上有所不同, 但基本结构是相同的. 您可以通过重写 `deploy` 任务来创建自己的流, 但更好的解决方案是使用缓存.

例如, 如果要在软链接新版本之前运行某些任务: 

~~~php
before('deploy:symlink', 'deploy:build');
~~~

或者, 在成功部署后发送通知:

~~~php
after('success', 'notify');
~~~

接下来快速了解一下每项任务.

### deploy:prepare

准备部署. 检查 `deploy_path` 是否存在, 否则创建它. 同时检查是否存在以下路径:

* `releases` – 在这个目录中, 将存储版本.
* `shared` – 跨所有版本共享文件.
* `.dep` – 部署程序使用的元数据.

### deploy:lock

部署锁, 只能运行一个并发部署. 此任务将检查 `.dep/deploy.lock` 文件. 如果部署过程被`Ctrl+C`取消, 请运行 `dep deploy:unlock` 删除此文件. 如果部署失败, 则 `deploy:unlock` 任务将自动触发. 

### deploy:release

基于 `release_name` 配置参数创建新的发布文件夹. 同时读取 `.dep/releases` 以获取以前创建的版本列表.

此外, 如果在 `deploy_path` 中有早期版本的符号链接, 则会将其删除.

### deploy:update_code

使用Git下载新版本的代码. 如果您使用的是Git 2.0版并且 `git_cache` 配置已打开, 则此任务将使用以前版本中的文件, 因此只下载更改的文件. 

在 `deploy.php` 中重写此任务, 创建自己的代码转移策略: 

~~~php
task('deploy:update_code', function () {
    upload('.', '{{release_path}}');
});
~~~

### deploy:shared

从 `shared` 目录创建共享文件和目录到 `release_path`. 您可以在 `shared_dirs` 和 `shared_files` 配置参数中指定共享目录和文件. 该过程分为以下步骤: 

* 指定的目录如果不存在将这些目录从 `release_path` 复制到`shared` 中, 
* 从 `release_path`中删除这些目录, 
* 软链接这些目录从`shared`到 `release_path`.

共享文件也遵循同样的步骤. 如果您的系统支持相对符号链接, 则将使用它们, 否则将使用绝对符号链接. 

### deploy:writable

设置 `writable_dirs` 参数中列出的目录可写. 默认情况下, 使用 `acl` 模式（使用`setfacl`命令）此任务将尝试猜测http_user用户名, 或者可以自己配置它: 
~~~php
set('http_user', 'www-data');

// Or only for specified host:
host(...)
    ->set('http_user', 'www-data');
~~~

此任务还支持其他可写模式:

* chown
* chgrp
* chmod
* acl

要使用其中一个, 请添加以下内容:

~~~php
set('writable_mode', 'chmod');
~~~

要将sudo与可写设置一起使用, 请添加以下内容:

~~~php
set('writable_use_sudo', true);
~~~

### deploy:vendors

安装 composer 依赖. 可以使用 `composer_options` 参数来配置composer选项. 

### deploy:clear_paths

删除 `clear_paths` 中指定的目录. 可以使用 `clear_use_sudo` 参数与sudo一起运行此任务. 

### deploy:symlink

将 `current` 软链接切换到 `release_path`. 如果目标系统支持符号链接的原子切换, 它将被使用. 

### deploy:unlock

删除`.dep/deploy.lock` 文件. 您可以直接运行此任务来删除锁文件: 

~~~sh
dep deploy:unlock staging
~~~

### cleanup

根据 `keep_releases` 选项来清理旧版本. `-1` 被视为不清理. 

### success

打印成功消息. 
