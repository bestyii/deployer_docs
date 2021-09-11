# 任务 (task)

使用`task`函数,定义个性化任务. 同时还可以使用`desc`函数设置任务说明:
```php
desc('My task');
task('my_task', function () {
    run(...);
});
```

执行任务:

```sh
dep my_task
```

列出全部可用的命令:

```sh
dep list
```

仅在指定的主机或环境(stage)上运行任务:

```sh
dep deploy main
```

你可以通过`--hosts`选项(多个值时,用英文逗号分隔)指定主机, 以及通过`--roles`选项指定角色:

```sh
dep deploy --hosts domain.com
dep deploy --roles app
```

### 简单任务

如果你的任务仅仅包含了`run`函数调用, 或只有一个bash命令, 可以简化任务定义:
```php
task('build', 'npm build');
```

> 默认情况下,所有的简单任务会先 cd 到 `release_path`目录下, 所以不重复做此步骤.

或者可以使用多行脚本:
 
```php
task('build', '
    gulp build;
    webpack -p;
    echo "Build done";
');
```

### 任务分组

你可以合并任务到一个分组中:

```php
task('deploy', [
    'deploy:prepare',
    'deploy:update_code',
    'deploy:vendors',
    'deploy:symlink',
    'cleanup'
]);
```

### Before 与 after

你可以定义任务在某些任务开始前或完成后执行.

``` php
task('deploy:done', function () {
    write('Deploy done!');
});

after('deploy', 'deploy:done');
```

在`deploy`任务调用之后, `deploy:done` 将会被执行.

### 过滤

您可以指定要在哪些主机(hosts)/环境(stage)/角色(roles)上运行任务。

### 按环境(stage)

按环境(stage)筛选主机:

``` php
desc('Run tests for application');
task('test', function () {
    ...
})->onStage('test');
```

### 按角色(roles)

按角色(roles)筛选主机:

``` php
desc('Migrate database');
task('migrate', function () {
    ...
})->onRoles('db');
```

还可以指定多个角色: `onRoles('app', 'db', ...)`.

### 按主机(hosts)名

按主机(hosts)名筛选主机:

``` php
desc('Migrate database');
task('migrate', function () {
    ...
})->onHosts('db.domain.com');
```

你还可以指定多个主机名:`onHosts('db.domain.com', ...)`.

### 本地任务

将任务标记为 `local` 以在本地运行，并且只运行一次，与主机计数无关。

```php
task('build', function () {
    ...
})->local();
```

>注意，在本地任务中调用 `run` 与调用`runLocally`具有相同的效果。 

### 仅运行一次

任务只运行一次:

```php
task('do', ...)->once();
```

将仅在第一台主机上运行.

### 重新配置

您可以重新配置任务，例如，按名称检索第三方recipes提供的任务：

```php
task('notify')->onStage('production');
```

### 任务复写

有时，您可能希望某些任务的行为与常见的方法不同。可以简单到覆盖它:
```php
task('deploy:update_code', function () {
    // Your custom update code
    upload(...);
});
```

### 使用输入选项

在定义任务之前，可以定义其他输入选项和参数:

``` php
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Input\InputArgument;

argument('stage', InputArgument::OPTIONAL, 'Run tasks only on this host or stage.');
option('tag', null, InputOption::VALUE_OPTIONAL, 'Tag to deploy.');
```

在任务内部使用这些:

``` php
task('foo:bar', function() {
    // 参数(arguments)
    $stage = null;
    if (input()->hasArgument('stage')) {
        $stage = input()->getArgument('stage');
    }
    
    // 选项(option)
    $tag = null;
    if (input()->hasOption('tag')) {
        $tag = input()->getOption('tag');
    }
});
```

### 任务并行执行

当部署到多个主机时，Deployer将在每个主机上运行一个任务:

<svg width="600" height="350" viewBox="0 0 600 350" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><g transform="translate(456 309)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(306 271)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(156 233)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(6 195)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="43" y="24">task 2</tspan></text></g><g transform="translate(456 157)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(306 119)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(156 81)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(6 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><path d="M3 35h594.5" stroke="#EBEBEB" stroke-linecap="square" stroke-dasharray="3,5"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="497" y="25">Host 4</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="347" y="25">Host 3</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="197" y="25">Host 2</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="47" y="25">Host 1</tspan></text></g></svg>

要加快部署, 请添加 `--parallel` 或 `-p` 选项. 这将在每个主机上并行运行任务. 如果在主机上执行任务所需的时间比其他主机长，Deployer将等待所有主机完成任务.

<svg width="600" height="153" viewBox="0 0 600 153" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><g transform="translate(456 91)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(306 91)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(156 91)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(6 91)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="43" y="24">task 2</tspan></text></g><g transform="translate(456 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(306 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(156 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(6 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><path d="M3 35h594.5" stroke="#EBEBEB" stroke-linecap="square" stroke-dasharray="3,5"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="497" y="25">Host 4</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="347" y="25">Host 3</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="197" y="25">Host 2</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="47" y="25">Host 1</tspan></text></g></svg>

通过指定一个数字来限制并发任务的数量. 默认情况下, 最多可同时处理10个任务.

```sh
dep deploy --parallel --limit 2
```

<svg width="600" height="210" viewBox="0 0 600 210" xmlns="http://www.w3.org/2000/svg"><g fill="none" fill-rule="evenodd"><g transform="translate(456 157)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(306 157)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(156 119)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="42" y="24">task 2</tspan></text></g><g transform="translate(6 119)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="43" y="24">task 2</tspan></text></g><g transform="translate(456 81)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(306 81)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(156 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><g transform="translate(6 43)"><rect fill="#EBEBEB" width="140" height="37.176" rx="8"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="41" y="24">task 1</tspan></text></g><path d="M3 35h594.5" stroke="#EBEBEB" stroke-linecap="square" stroke-dasharray="3,5"/><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="497" y="25">Host 4</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="347" y="25">Host 3</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="197" y="25">Host 2</tspan></text><text font-family="Monaco" font-size="16" fill="#9B9B9B"><tspan x="47" y="25">Host 1</tspan></text></g></svg>

下一节: [主机](hosts.md).
