# 主机 (Hosts)

在Deployer中定义主机是部署应用所必需的. 它可以是远程计算机、本地计算机或云主机实例.
每个主机都包含主机名(hostname、环境(stage)、一个或多个角色(roles)以及配置参数.

在 `deploy.php` 文件中使用 `host` 函数定义主机。以下是主机定义的示例：
~~~php
host('domain.com')
    ->stage('production')
    ->roles('app')
    ->set('deploy_path', '~/app');
~~~

主机 *domain.com* 包含 `production`(生产) 环境(stage)定义, 一个 `app` 角色以及一个配置参数 `deploy_path` = `~/app`.

主机还可以用yaml语法来描述. 把下列定义保存到文件 `hosts.yml`:
~~~yaml
domain.com:
  stage: production
  roles: app
  deploy_path: ~/app
~~~

然后在 `deploy.php`中引用:

~~~php
inventory('hosts.yml');
~~~

确保`~/.ssh/config`文件包含有关连接信息.
或者您可以在 `deploy.php` 文件本身中定义.

~~~php
host('domain.com')
    ->user('name')
    ->port(22)
    ->configFile('~/.ssh/config')
    ->identityFile('~/.ssh/id_rsa')
    ->forwardAgent(true)
    ->multiplexing(true)
    ->addSshOption('UserKnownHostsFile', '/dev/null')
    ->addSshOption('StrictHostKeyChecking', 'no');
~~~

> **最佳实践** 在文件 `~/.ssh/config` 中保存相关连接信息.
> 这样就允许不同的用户以不同的方式进行连接.

### 主机配置覆盖

例如，如果您有一些全局配置, 则可以按主机覆盖它:

~~~php
set('branch', 'master');

host('prod')
    ...
    ->set('branch', 'production');
~~~

这样在  _prod_ 主机上分支设置成了 `production`, 其他主机默认是 `master`.

### 获取主机信息

在任何任务中, 可以通过`get`函数获取主机配置, 通过`host`函数获取主机对象.
~~~php
task('...', function () {
    $deployPath = get('deploy_path');
    
    $host = host('domain.com');
    $port = $host->getPort();
});
~~~

### 多主机

可以传多个主机参数给`host`函数:
~~~php
host('110.164.16.59', '110.164.16.34', '110.164.16.50', ...)
    ->stage('production')
    ...
~~~

如果你主机列表文件`hosts.yml`包含多个主机，您可以用相同的方式更改所有的配置.

~~~php
inventory('hosts.yml')
    ->roles('app')
    ...
~~~

### 主机范围

如果有许多主机遵循类似的模式,可以这样描述它们,而不是列出每个主机名：

~~~php
host('www[01:50].domain.com');
~~~

对于数字模式，可以根据需要包含或删除补位的`0`.

您还可以定义字母范围：

~~~php
host('db[a:f].domain.com');
~~~

### 本地主机 (Localhost)

如果您需要在部署到远程主机之前构建发行版，或者部署到localhost而不是remote，
您需要定义localhost:
~~~php
localhost()
    ->stage('production')
    ->roles('test', 'build')
    ...
~~~

### 主机别名 (Host aliases)

如果要将应用程序部署到同一个主机(例如在不同的目录中), 可以描述两个主机别名:
~~~php
host('domain.com/green', 'domain.com/blue')
    ->set('deploy_path', '~/{{hostname}}')
    ...
~~~

这对于Deployer, 相当于两个主机, 部署后您将看到以下目录结构：

~~~
~
└── domain.com
    ├── green
    │   └── ...
    └── blue
        └── ...
~~~

### 同主机下多环境(stage)

有时, prod和beta环境只有一台服务器. 您可以轻松地配置它们:
~~~php
host('production')
    ->hostname('domain.com')
    ->set('deploy_path', '~/domain.com');
    
host('beta')
    ->hostname('domain.com')
    ->set('deploy_path', '~/beta.domain.com');    
~~~

可以使用以下命令进行部署：

~~~sh
dep deploy production
dep deploy beta
~~~

### 主机清单文件

通过 `inventory` 函数引入主机清单文件`hosts.yml`：

~~~php
inventory('hosts.yml');
~~~

下面是一个带有全套配置的主机清单示例文件 `hosts.yml`
~~~yaml
domain.com:
  hostname: domain.com
  user: name
  port: 22
  configFile: ~/.ssh/config
  identityFile: ~/.ssh/id_rsa
  forwardAgent: true
  multiplexing: true
  sshOptions:
    UserKnownHostsFile: /dev/null
    StrictHostKeyChecking: no
  stage: production
  roles:
    - app
    - db
  deploy_path: ~/app
  extra_param: "foo {{hostname}}"
~~~

> **注意** 就像在 *deploy.php* 文件中通过 `host` 函数定义主机时的建议, 最好省略如: *user*、 *port*、 *identityFile*、 *forwardAgent* 相关参数, 并在文件 `~/.ssh/config` 中设置.

如果清单文件中定义了多个主机, 可以使用YAML的扩展语法:
~~~yaml
.base: &base
  roles: app
  deploy_path: ~/app
  ...

www1.domain.com:
  <<: *base
  stage: production
  
beta1.domain.com:
  <<: *base
  stage: beta
    
...
~~~

以`.`（*点*）开头的主机称为隐藏主机，在该文件外不可见。 

在清单文件中添加`local`声明:
~~~yaml
localhost:
  local: true
  roles: build
  ...
~~~

### Become

Deployer允许您 '成为(become)' 另一个用户，该用户有别与登录到计算机的用户(远程用户).

~~~php
host('domain.com')
    ->become('deployer')
    ...
~~~

默认情况下，Deployer使用 `sudo` 权限提升方法。

> **注意** 这个become在`tty`时无效.

下一节: [流水线](flow.md).
