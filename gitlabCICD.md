# Gitlab CI/CD 介绍和使用

## 一、概念

1. 持续集成(Continuous Integration)：频繁地(一天多次)将代码集成到主干。让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。“持续集成并不能消除 Bug，而是让它们非常容易发现和改正。”
2. 持续交付(Continuous Delivery)：频繁地将软件的新版本，交付给质量团队或者用户，以供评审。如果评审通过，代码就进入生产阶段。持续交付可以看作持续集成的下一步。它强调的是，不管怎么更新，软件是随时随地可以交付的。
3. 持续部署(continuous Deployment)：代码通过评审以后，自动部署到生产环境。是持续部署是持续交付的下一步，持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。

## 二、Gitlab持续集成

### 概念

1. Gitlab：是一个利用Ruby on Rails开发的开源应用程序，实现一个自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。它拥有与GitHub类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。
2. Gitlab CI/CD：GitLab Continuous Integration（Gitlab持续集成）的简称。GitLab 自GitLab 8.0开始提供了持续集成的功能，且对所有项目默认开启。只要在项目仓库的根目录添加.gitlab-ci.yml文件，并且配置了Runner（运行器），那么每一次push或者合并请求（Merge Request）都会触发CI Pipeline。
3. Gitlab Runner：可以运行在 GNU / Linux，macOS 和 Windows 操作系统上。每次push的时候 GitLab CI 会根据.gitlab-ci.yml配置文件运行你流水线（Pipeline）中各个阶段的任务（Job），并将结果发送回 GitLab。GitLab Runner 是基于 Gitlab CI 的 API 进行构建的相互隔离的机器（或虚拟机）。GitLab Runner 不需要和 Gitlab 安装在同一台机器上，且考虑到 GitLab Runner 的资源消耗问题和安全问题，也不建议这两者安装在同一台机器上。Gitlab分为三种：共享Runner、专享Runner和分组Runner。
4. Pipelines：是分阶段执行的构建任务。如：安装依赖、运行测试、打包、部署开发服务器、部署生产服务器等流程。每一次push或者Merge Request都会触发生成一条新的Pipeline。
5. Stages：构建阶段，可以理解为上面所说“安装依赖”、“运行测试”等环节的流程。我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：1. 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始（当然可以在.gitlab-ci.yml文件中配置上一阶段失败时下一阶段也执行）；2. 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功；3. 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败。
6. Jobs：表示构建的作业（或称之为任务），表示某个 Stage 里面执行的具体任务。我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：1. 相同 Stage 中的 Jobs 无执行顺序要求，会并行执行；2. 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功；3. 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 也失败（可以在`.gitlab-ci.yml`文件中配置允许某 Job 可以失败，也算该 Stage 成功）。
7. `.gialab-ci.yml`:GitLab 中默认开启了 Gitlab CI/CD 的支持，且使用YAML文件`.gitlab-ci.yml`来管理项目构建配置。该文件需要存放于项目仓库的根目录（默认路径，可在 GitLab 中修改），它定义该项目的 CI/CD 如何配置。所以，我们只需要在`.gitlab-ci.yml`配置文件中定义流水线的各个阶段，以及各个阶段中的若干作业（任务）即可。
简单的`.gitlab-ci.yml`文件实例：

```yaml
# 定义 test 和 package 两个 Stages
stages:
  - test
  - package

# 定义 package 阶段的一个 job
package-job:
  stage: package
  script:
    - echo "Hello, package-job"
    - echo "I am in package stage"

# 定义 test 阶段的一个 job
test-job:
  stage: test
  script:
    - echo "Hello, test-job"
    - echo "I am in test stage"
```

以上配置中，用 stages 关键字来定义 Pipeline 中的各个构建阶段，然后用一些非关键字来定义 jobs。每个 job 中可以可以再用 stage 关键字来指定该 job 对应哪个 stage。job 里面的script关键字是每个 job 中必须要包含的，它表示每个 job 要执行的命令。

### Gitlab CI/CD yaml常用配置

开始构建之前`.gitlab-ci.yml`文件定义了一系列带有约束说明的任务。这些任务都是以任务名开始并且至少要包含script部分，`.gitlab-ci.yml`允许指定无限量 jobs。每个 jobs 必须有一个唯一的名字，且名字不能是下面列出的保留字段：

* `image`
* `services`
* `stages`
* `types`
* `before_script`
* `after_script`
* `variables`
* `cache`

job由一列参数来定义 jobs 的行为：
|关键字|必需|功能|
|--|--|---|
|script|Y|Runner执行的命令或脚本|
|extends|N|定义此作业将继承的配置条目|
|image|N|所使用的docker镜像|
|services|N|所使用的docker服务|
|stage|N|定义job stage（默认：`test`）|
|type|N|`stage`的别名|
|varibles|N|定义job级别的变量|
|only|N|定义一列git分支，并为其创建job|
|except|N|定义一列git分支，不创建job|
|tags|N|定义一列tags，用来指定选择哪个Runner（同时Runner也要设置tags）|
|allow_failure|N|允许job失败。失败的job不影响commit状态|
|when|N|定义何时开始job。可以是`on_success`，`on_failure`，`always`或者`manual`|
|dependencies|N|定义job依赖关系，这样他们就可以互相传递artifacts|
|cache|N|定义应在后续运行之间缓存的文件列表|
|before_script|N|重写一组在作业前执行的命令|
|after_script|N|重写一组在作业后执行的命令|
|environment|N|定义此作业完成部署的环境名称|
|coverage|N|定义给定作业的代码覆盖率设置|
|etry|N|定义在发生故障时可以自动重试作业的时间和次数|
|parallel|N|定义应并行运行的作业实例数|

## YAML 教程

## 基本语法

* 大小写敏感
* 使用缩进表示层级关系
* 缩进不允许使用tab，只允许空格
* 缩进的空格数不重要，只要相同层级的元素左对齐即可
* '#'表示注释

## 数据类型

Yaml 支持以下几种数据类型：

* 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
* 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
* 纯量（scalars）：单个的、不可再分的值

## YAML 对象

对象键值对使用冒号结构表示 key: value，冒号后面要加一个空格。也可以使用 key:{key1: value1, key2: value2, ...}。还可以使用缩进表示层级关系；

```yaml
key: 
    child-key: value
    child-key2: value2
```

## YAML 数组

以`-`开头的行表示构成一个数组：

```yaml
- A
- B
- C
```

数据结构的子成员是一个数组，则可以在该项下面缩进一个空格:

```yaml
-
 - A
 - B
 - C
```

一个相对复杂的例子：

```yaml
companies:
    -
        id: 1
        name: company1
        price: 200W
    -
        id: 2
        name: company2
        price: 500W
```

意思是 companies 属性是一个数组，每一个数组元素又是由 id、name、price 三个属性构成。

## 复合结构

数组和对象可以构成复合结构，例：

```yaml
languages:
  - Ruby
  - Perl
  - Python 
websites:
  YAML: yaml.org 
  Ruby: ruby-lang.org 
  Python: python.org 
  Perl: use.perl.org
```

转换为json为：

```json
{ 
  languages: [ 'Ruby', 'Perl', 'Python'],
  websites: {
    YAML: 'yaml.org',
    Ruby: 'ruby-lang.org',
    Python: 'python.org',
    Perl: 'use.perl.org' 
  } 
}
```

## 纯量

```yaml
oolean: 
    - TRUE  #true,True都可以
    - FALSE  #false，False都可以
float:
    - 3.14
    - 6.8523015e+5  #可以使用科学计数法
int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示
null:
    nodeName: 'node'
    parent: ~  #使用~表示null
string:
    - 哈哈
    - 'Hello world'  #可以使用双引号或者单引号包裹特殊字符
    - newline
      newline2    #字符串可以拆成多行，每一行会被转化成一个空格
date:
    - 2018-02-17    #日期必须使用ISO 8601格式，即yyyy-MM-dd
datetime: 
    -  2018-02-17T15:02:31+08:00    #时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区
```
