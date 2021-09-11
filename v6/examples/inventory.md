# 主机清单模版 (Inventory Examples)

你可以从下列例子中选择适合你的方式.

### 主机数量不多的时候

在大多数情况下, 一般项目会有一个或两个主机: 一个用于生产, 另一个用于其他阶段.

所以不需要独立的主机清单文件, 把全部的配置直接写到 _deploy.php_ 文件中.

对于单个主机. Deployer 不需要指定 _stage_ 参数.

```php
set('deploy_path', '~/project');

host('project.com');
```

如果你有两台主机,如:一个测试一个生产, 下面这些配置就能满足.

> 由于设置了 _default_stage_ 参数,所以 `dep deploy` 命令 , 部署的是 _staging_. 真正部署生产环境的命令是 `dep deploy production`.

```php
set('application', 'project');
set('deploy_path', '~/{{application}}');
set('default_stage', 'staging');

host('project.com')
    ->stage('production');
    
host('staging.project.com')
    ->stage('staging');
```

> **最佳实践** 在文件 `~/.ssh/config` 中保存相关连接信息.
> 这样就允许不同的用户以不同的方式进行连接.

### 剥离到独立的主机清单文件中

TODO
