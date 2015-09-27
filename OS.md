# OS#


### 稳定度: 2 - 稳定

提供一些基本的操作系统相关的功能。

使用`require('os')`来获得这个模块。

#### os.tmpdir()#

返回操作系统默认的临时文件目录。

#### os.homedir()#

返回当前用户的家目录。

#### os.endianness()#

返回CPU的字节序。`BE`为大端字节序，`LE`为小端字节序。

#### os.hostname()#

返回当前操作系统的主机名。

#### os.type()#

返回操作系统名。例如，Linux下为`'Linux'`，OS X下为`'Darwin'`，Windows下为`'Windows_NT'`。

#### os.platform()#

返回操作系统平台。可能的值有`'darwin'`，`'freebsd'`，`'linux'`，`'sunos'`或`'win32'`。返回`process.platform`值。

#### os.arch()#

返回操作系统CPU架构。可能的值有`'x64'`，`'arm'`和`'ia32'`。返回`process.arch`值。

#### os.release()#

返回操作系统的发行版本。

#### os.uptime()#

返回操作系统的运行时间（秒）。

#### os.loadavg()#

返回一个包含1，5，15分钟平均负载的数组。

平均负载是一个系统活动测量，由操作系统计算并且由一个分数表示。根据经验，理想的负载均衡数应该比系统的逻辑CPU数小。

平均负载完全是一个`UNIX-y`概念；在Windows中没有完全对等的概念。所以在Windows下，这个函数总是返回`[0, 0, 0]`。

#### os.totalmem()#

以字节的形式返回系统的总内存。

#### os.freemem()#

以字节的形式返回系统的可用内存。

#### os.cpus()#

返回一个包含安装的各个CPU/核心信息的对象数组：型号，速度（单位MHz），和时间（一个包含CPU/核心花费的毫秒数的对象：`user`，`nice`，`sys`，`idle`和`irq`）。

`os.cpus`例子：

```js
[ { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 252020,
       nice: 0,
       sys: 30340,
       idle: 1070356870,
       irq: 0 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 306960,
       nice: 0,
       sys: 26980,
       idle: 1071569080,
       irq: 0 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 248450,
       nice: 0,
       sys: 21750,
       idle: 1070919370,
       irq: 0 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 256880,
       nice: 0,
       sys: 19430,
       idle: 1070905480,
       irq: 20 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 511580,
       nice: 20,
       sys: 40900,
       idle: 1070842510,
       irq: 0 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 291660,
       nice: 0,
       sys: 34360,
       idle: 1070888000,
       irq: 10 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 308260,
       nice: 0,
       sys: 55410,
       idle: 1071129970,
       irq: 880 } },
  { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
    speed: 2926,
    times:
     { user: 266450,
       nice: 1480,
       sys: 34920,
       idle: 1072572010,
       irq: 30 } } ]
```

注意因为`nice`值是UNIX中心的，所以在Windows中所有进程的`nice`值都将是`0`。

#### os.networkInterfaces()#

获取一个网络接口列表：

```js
{ lo:
   [ { address: '127.0.0.1',
       netmask: '255.0.0.0',
       family: 'IPv4',
       mac: '00:00:00:00:00:00',
       internal: true },
     { address: '::1',
       netmask: 'ffff:ffff:ffff:ffff:ffff:ffff:ffff:ffff',
       family: 'IPv6',
       mac: '00:00:00:00:00:00',
       internal: true } ],
  eth0:
   [ { address: '192.168.1.108',
       netmask: '255.255.255.0',
       family: 'IPv4',
       mac: '01:02:03:0a:0b:0c',
       internal: false },
     { address: 'fe80::a00:27ff:fe4e:66a1',
       netmask: 'ffff:ffff:ffff:ffff::',
       family: 'IPv6',
       mac: '01:02:03:0a:0b:0c',
       internal: false } ] }
```

注意，由于底层系统实现的原因，它将只会返回被赋予一个地址的网络接口。

#### os.EOL#

一个定义了对于操作系统，合适的行结束记号的常量。
