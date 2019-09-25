---
title: 前端初探 Gitlab CI/CD
date: 2019-09-25 21:58:51
categories: [技术, 前端]
tags: [Gitlab, CI/CD]
---

## 前言

纵观人类历史的发展以及三次工业革命，你会发现利用机器来替代部分人力劳动，将重复的工作自动化从而解放生产力都是发展的必然趋势，在软件工程领域也不例外，其中 CI/CD 就是其中一项，那么什么是 CI/CD 呢，网上的解释不要太多，这里我就直接放一幅 Gitlab 官网的工作流程图好了：

<!-- more -->

![](https://user-gold-cdn.xitu.io/2019/9/25/16d66a2af2890c40?w=3420&h=1894&f=png&s=70061)

## 准备条件

1. [Gitlab runner](https://docs.gitlab.com/ee/ci/runners/README.html)
2. [.gitlab-ci.yml](https://docs.gitlab.com/ee/ci/yaml/README.html)

## Gitlab runner

Gitlab runner 是整个 CI/CD 的执行器，它是执行你写的 .gitlab-ci.yml 文件的虚拟机。  
Gitlab runner 分为两种：

1. 特定的 runner：只能当前项目使用
2. 共享的 runner：所有项目都可以使用

找到你的项目在 **设置**>**Runners** 里  
你可以看到如下界面：

![](https://user-gold-cdn.xitu.io/2019/9/25/16d6708355b4c2f1?w=1308&h=467&f=png&s=52910)
左侧就是特定的 runners 右侧就是共享的 runners，只要确保有其一就行。  
关于 runner 的安装我不想过多赘述，官网写的很清楚，只要按照步骤一步一步搭建就好了。

## .gitlab-ci.yml

当你有了 runner 就可以开始着手写 .gitlab-ci.yml 文件了，.gitlab-ci.yml 文件是对于整个 CI/CD 流程的描述文件，它告诉 runner 应该怎样执行具体的操作。  
在具体介绍配置之前，我想先明确整个 .gitlab-ci.yml 里面的几个重要名词：

1. job: job 是整个 CI/CD 的核心单元，它定义了在什么条件下应该执行什么任务，每个 job 都是相互隔离的。
2. script: job 下的属性，用于描述 job 要执行的任务。
3. stages: 定义 job 的分组，不同 job 可以所属于不同的阶段，一共有三个阶段可供选择：test build deploy，stage 的执行是按顺序的，但是 stage 下面的 job 是并行执行的，只有前一个 stage 执行成功才会执行下一个 stage，一旦上一个 stage 中任何一个 job 执行失败都会导致整个流水线失败。
4. pipeline: 上面的整个流程就是一个流水线。

下面是我的一个前端项目的 .gitlab-ci.yml 文件，以它来作为示例：

```yaml
image: node:10.13

stages:
  - test
  - build
  - deploy

cache:
  paths:
    - node_modules/

before_script:
  ## set proxy
  - export http_proxy=http://10.2.3.63:3128/
  - export https_proxy=http://10.2.3.63:3128/

test:
  stage: test
  tags:
    - sams
  script:
    - npm install --no-optional --registry=https://registry.npm.taobao.org
    - npm run lint
  only:
    - master
    - dev

build:
  stage: build
  tags:
    - sams
  script:
    - npm run build
  artifacts:
    paths:
      - $SOURCE_DIR
    expire_in: 2 mins
  only:
    - master

deploy:
  stage: deploy
  tags:
    - sams
  before_script:
    ## set debian mirros
    - echo 'deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib' > /etc/apt/sources.list
    - echo 'deb-src http://mirrors.aliyun.com/debian/ stretch main non-free contrib' >> /etc/apt/sources.list
    - echo 'deb http://mirrors.aliyun.com/debian-security stretch/updates main' >> /etc/apt/sources.list
    - echo 'deb-src http://mirrors.aliyun.com/debian-security stretch/updates main' >> /etc/apt/sources.list
    - echo 'deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib' >> /etc/apt/sources.list
    - echo 'deb-src http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib' >> /etc/apt/sources.list
    - echo 'deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib' >> /etc/apt/sources.list
    - echo 'deb-src http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib' >> /etc/apt/sources.list
    ## Using SSH keys with GitLab CI/CD
    ## https://docs.gitlab.com/ee/ci/ssh_keys/README.html
    - "which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )"
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - echo "$SSH_KNOWN_HOSTS" > ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
  script:
    - scp -r $SOURCE_DIR $DEPLOY_SERVER_USER@$DEPLOY_SERVER_IP:$TARGET_DIR
  only:
    - master
  environment: test
```

### 全局配置

- image: 要使用的 docker 镜像它会根据你写的镜像名从 [docker hub](https://hub.docker.com/) 上面拉取镜像，因为我们的项目是前端项目，所以这里配置的镜像是 node，最终你的 script 指定的脚本会跑在 node 的 docker 环境下，这样就保证你有了必要的环境依赖（node）
- stages: 定义整个流水线的各个阶段，这里我三个阶段都定义了，但是你可以根据你自己项目的实际情况定义，定义好后流水线将按照你定义的顺序依次执行。
- cache: 定义 job 之间要缓存的文件，通常我们都会把项目的安装依赖作为缓存，这样下一个 job 就不用重新安装依赖了
- before_script: 定义所有 job script 执行前需要执行的脚本，这里我设置了代理。

### job 的配置

- stage: 定义该 job 所属的 stage
- tags: 指定该 job 使用的 runner
- script: 该 job 需要执行的脚本
- only: 该 job 的约束条件，你可以指定该 job 在哪些情况下会触发，比如只有 master 分支和 develop 分支才会执行 deploy 的 job，与此相对的还有一个 except 属性表示什么条件下不执行该 job
- artifacts: 用于在不同 stage 之间传递结果，通用的做法是将 build 阶段打包出来的文件定义为 artifacts，这样在 deploy 阶段就可以直接使用了，这里你可能会对 artifacts 和 cache 有些搞不清，这里推荐看官方的[说明](https://docs.gitlab.com/ee/ci/caching/#cache-vs-artifacts)
- environment: 用于定义 job 部署的环境，这里需要结合 Gitlab 项目的 UI 界面，比如这里我设置的名称叫 test，每次部署成功之后 test 环境下就会多一个部署条目，你可以重新部署甚至回滚到某个部署，如下图所示：

![](https://user-gold-cdn.xitu.io/2019/9/25/16d66e200a793224?w=1309&h=516&f=png&s=58035)

> 另外 environment 下还有一个 url 属性可以定义部署到的服务器地址，这样你可以在 UI 界面通过点击按钮直接跳转到项目，如果你的 Gitlab 低于 8.11 那只能通过在 UI 界面（上图）手动配置了。

### ssh keys 实现免密登录

一般来讲你部署项目的时候不可避免会用到 ssh 协议，但是 ssh 协议需要你手动输入用户名和密码，这样不就无法实现自动化部署了？ 别急，Gitlab 已经帮我们想到了这一点，仔细阅读官网上的 [Using SSH keys with GitLab CI/CD](https://docs.gitlab.com/ee/ci/ssh_keys/README.html#using-ssh-keys-with-gitlab-cicd) 这篇文章你就能找到解决方案，或者参考我的实例代码：

```yaml
## Using SSH keys with GitLab CI/CD
## https://docs.gitlab.com/ee/ci/ssh_keys/README.html
- "which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )"
- eval $(ssh-agent -s)
- echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
- mkdir -p ~/.ssh
- chmod 700 ~/.ssh
- echo "$SSH_KNOWN_HOSTS" > ~/.ssh/known_hosts
- chmod 644 ~/.ssh/known_hosts
```

这里我还是要简单说一下其实现原理，Gitlab 首先让你生成一对 ssh 的公钥和私钥，在每次执行 CI/CD 时将私钥（此时已添加到环境变量里）通过 ssh-add 添加到 ssh-agent 里面进行管理，并将设置好的 \$SSH_KNOWN_HOSTS (里面就包含了你要 shh 的主机) 也添加到 known_hosts 文件中，这样在执行 ssh 命令时远程机器就能识别你的身份从而实现免密登录，本质上和你本地实现免密登录的道理是一样的。

## 其它

### 环境变量

类似于编程中的变量，环境变量可以存储一些要要变化的或是比较私密的信息，环境变量配置好后可以通过 **\$ + 变量名** 在 script 里面进行调用
![](https://user-gold-cdn.xitu.io/2019/9/25/16d66f4c0044aa11?w=1308&h=457&f=png&s=43890)

### 查看流水线状态

在 Gitlab 主页找到流水线页签，打开之后就能看到所有流水线的运行情况
![](https://user-gold-cdn.xitu.io/2019/9/25/16d66f72c5a05bb2?w=1329&h=438&f=png&s=53552)
点击进入具体的 stage 你可以看到其执行细节，如果失败也可以重新执行
![](https://user-gold-cdn.xitu.io/2019/9/25/16d66f8ee9d09638?w=1351&h=521&f=png&s=62355)

## 参考资料

https://docs.gitlab.com/ee/ci/README.html

## 总结

这次也是我初次尝试使用 Gitlab CI/CD ，每次任务大概执行需要 8 分钟，如果你像之前手动去做这些工作那你每次都至少需要花费 8 分钟时间，不要小瞧这 8 分钟，日积月累也是一笔不小的开支，更重要的是机器很少出错的，但是人的话就没法保证了，而且就像我在开篇所说简单重复性的工作必然会被机器取代，这是不可避免的历史规律，不管你用不用它，技术的潮流都会不断向前推动，所以还不如提前拥抱它。  
另外此次只介绍了整个 Gitlab CI/CD 功能的冰山一角，Gitlab 还提供很多优秀的功能，比如在 merge request 的时候进行 CI/CD，所以更多的功能还有待挖掘，如果你感兴趣可以去官方文档上找寻，如果有什么不对的地方还请您指正，最后感谢您的阅读！
