# API 参考资料

### host

* `host(string ...$hostname): Host`

声明一个主机或一组主机. 了解更多请移步 [主机](hosts.md).

### localhost

* `localhost(string ...$alias = 'localhost'): Host`

声明本地主机.

### inventory

* `inventory(string $file): Host[]`

载入主机清单文件

### desc

* `desc(string $description)`

设置任务描述.

### task

* `task(string $name, string $script)`
* `task(string $name, callable $callable)`
* `task(string $name): Task`

声明一个任务或获取一个任务. 了解更多请移步 [任务](tasks.md).

### before

* `before(string $when, string $that)`

在任务 `$when` 之前, 执行任务 `$that`.

### after

* `after(string $when, string $that)`

在任务 `$when` 之后, 执行任务`$that`.

### fail

* `fail(string $what, string $that)`

如果任务 `$what` 执行失败, 执行任务 `$that`.

### argument

* `argument($name, $mode = null, $description = '', $default = null)`

添加用户的cli参数.

### option

* `option($name, $shortcut=null, $mode=null, $description='', $default=null)`

添加用户的cli选项.

### cd

* `cd(string $path)`

设置`run`函数下的工作路径.
每个任务都会将工作路径恢复到任务开始时的基本工作路径. 

~~~php
cd('{{release_path}}');
run('npm run build');
~~~

### within

* `within(string $path, callable $callback)`

在指定的路径 `$path`内部运行回调函数`$callback`.

~~~php
within('{{release_path}}', function () {
    run('npm run build');   
});
~~~

### workingPath

* `workingPath(): string`

返回当前工作路径. 


~~~php
cd('{{release_path}}');
workingPath() == '/var/www/app/releases/1';
~~~

### run

* `run(string $command, $options = []): string`

在远程主机上运行命令. 可用选项:

* `timeout` — 设置进程超时 (最大运行时间) .
  要禁用超时, 请将此值设置为`null`.  
  超时时间(秒) 默认值:300秒
* `tty` — 启用或禁用TTY模式 默认值:false

例如, 如果您的私钥包含密码短语, 启用tty, 您将看到git提示输入密码. 

~~~php
run('git clone ...', ['timeout' => null, 'tty' => true]);
~~~

`run`函数以字符串形式返回输出的命令:
   
~~~php
$path = run('readlink {{deploy_path}}/current');
run("echo $path");
~~~

### runLocally

* `runLocally($command, $options = []): string`

在localhost上运行命令. 可用选项:

* `timeout` — 超时时间(秒) 默认值:300秒
* `tty` — TTY模式 默认值:false

### test

* `test(string $command): bool`

运行测试命令.
 
~~~php
if (test('[ -d {{release_path}} ]')) {
    ...
}
~~~

### testLocally

* `testLocally(string $command): bool`

在本地运行测试命令.

### on

* `on(Host $host, callable $callback)`
* `on(Host[] $host, callable $callback)`

在指定的主机上运行函数 `$callback` .

~~~php
on(host('domain.com'), function ($host) {
   ...
});
~~~

~~~php
on(roles('app'), function ($host) {
   ...
});
~~~

~~~php
on(Deployer::get()->hosts, function ($host) {
   ...
});
~~~

### roles

* `roles(string ...$role): Host[]`

按角色返回主机列表.

### invoke

* `invoke(string $task)`

在当前主机上运行任务. 

~~~php
task('deploy', function () {
    invoke('deploy:prepare'); 
    invoke('deploy:release');
    ...
});
~~~

> **注意** 这个是实验功能.

### upload

* `upload(string $source, string $destination, $config = [])`

从 `$source` 上传文件到远程主机 `$destination` .

~~~php
upload('build/', '{{release_path}}/public');
~~~

> 您可能已经注意到, 在上述命令的第一个参数的末尾有一个斜杠（/）, 
> 意思就是 "`build`下的内容".
>
> 另一种方法是不使用斜杠, 将 `build`这个目录直接放到 `public`中. 
> 他的创建的层级, 是这个样子: `{{release_path}}/public/build`

可用选项:

* `timeout` — 超时时间 默认值: null
* `options` — `rsync` 选项.

### download

* `download(string $source, string $destination, $config = [])`

从远程主机 `$source` 下载文件到本地主机 `$destination` 中.

可用选项:

* `timeout` — 超时时间 默认值: null
* `options` — `rsync` 选项.

### write

在输出中写入消息.
您可以使用标记格式化消息 `<info>...</info>`, `<comment></comment>` or `<error></error>` (参考 [Symfony Console](http://symfony.com/doc/current/console/coloring.html)).

### writeln

与 `write`函数相同, 但会另起一行. 

### set

* `set(string $name, string|int|bool|array $value)`
* `set(string $name, callable $value)`

设置全局配置参数. 如果callable作为`$value`传递, 它将在第一次获取此配置时触发. 

了解更多请移步 [配置](configuration.md).

### add

* `add(string $name, array $values)`

向现有配置添加值. 

了解更多请移步 [配置](configuration.md).

### get

* `get(string $name, $default = null): string|int|bool|array`

获取配置值.

了解更多请移步 [配置](configuration.md).

### has

* `has(string $name): bool`

检查配置项是否存在. 


了解更多请移步 [配置](configuration.md).

### ask

* `ask(string $message, $default = null, $suggestedChoices = null)`

请求用户输入. 


### askChoice

* `askChoice(string $message, array $availableChoices, $default = null, $multiselect = false)`

要求用户从多个键/值选项中选择并返回一个数组. 
启用多选时, 结果使用逗号隔开选中的内容.
默认值将在静默模式下使用, 否则将接受第一个可用选项. 

### askConfirmation

* `askConfirmation(string $message, bool $default = false)`

询问用户 “是” 或 “否” 的问题 .

### askHiddenResponse

* `askHiddenResponse(string $message)`

询问用户密码.

### input

* `input(): Input`

获取当前控制台输入.

### output

* `output(): Output`

获取当前控制台输出.

### isQuiet

* `isQuiet(): bool`

检查 `dep` 命令是否用 `-q` 选项启动. 

### isVerbose

* `isVerbose(): bool`

检查 `dep` 命令是否用 `-v` 选项启动.

### isVeryVerbose

* `isVeryVerbose(): bool`
  
检查 `dep` 命令是否用 `-vv` 选项启动.

### isDebug

* `isDebug(): bool`

检查 `dep` 命令是否用 `-vvv` 选项启动.

### commandExist

* `commandExist(string $command): bool`

检查命令是否存在.

~~~php
if (commandExist('composer')) {
    ...
}
~~~

### parse

* `parse(string $line): string`

解析在配置 `$line` 中出现的 `{{` `}}`.
