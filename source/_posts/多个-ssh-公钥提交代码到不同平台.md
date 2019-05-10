---
title: 多个 ssh 公钥提交代码到不同平台
date: 2019-04-17 22:09:21
tags: Git
---


# 多个 SSH 公钥提交代码到不同平台

作为一个技术开发人员，免不了在 `github` 以及 `gitlab` 以及其他 `git` 平台上进行代码管理；
工作中您可能使用 `svn` （这不在我们的讨论范围）,也有可能使用 `git`, 生活中，您可能会将一些内容分享到你的 `github` 上， 供大家参阅。

`git` 创建版本库很容易， `clone` 代码也仅仅是简单的一句 `git clone https://github.com/xxxx.git`，异或是 `git clone git@github.com:xxxxx.git`；当然，使用 `https` 的方式简单易操作（本人认为是一个傻瓜相机），但是通过使用 `https` 的方式，经常会遇到需要输入账号和密码的情况，这大大的加大了安全问题，虽然某 dear 的图形化工具，会帮助我们 remeber 账号和密码，避免了重复输入，但这不在我们的讨论范围（个人很鄙视使用图形化界面的），接下来要说的就是使用 `SSH` 的方式来处理 `git` 的版本管理。

<!--more-->

## 生成 SSH 密钥

使用 `SSH` 创建一套公密钥，将公钥添加到你要使用的 `Git` 平台账户下

```
ssh-keygen -t rsa -C "your email addr" -f ~/.ssh/github
```

注意：
1. `-f` 后面的参数是用来自定义 SSH KEY 的存放路径，如果不需要也可以自 -f 开始省略
2. 命令输入完成后，连击3下回车就可以，不需要处理操作（除非你很想处理）

## 添加生成的 SSH 公钥

添加 ssh 公钥到 `github` **
    
1. 打开 `https://github.com/settings/profile` ，选择 `SSH and GPG keys`

    ![](https://raw.githubusercontent.com/boywithsmalleyes/static_file/master/images/%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE_%E9%80%89%E6%8B%A9%E5%8C%BA%E5%9F%9F_20190425135335.png)
2. 点击 `New SSH key`

    1. title 可以随便写，建议见名知意，能知道是哪台设备
    2. key 通过刚刚通过 `SSH` 命令生成的 `.pub` 文件中复制即可。文件路径 `.ssh/id_rsa.pub` 异或是存在您 `-f` 之后指定的目录。
    3. 点击 `add SSH key`


## 配置多个 ssh

配置多个 `ssh` 时，需要注意的是：

1. 如果你未指定公钥的存储路径，那么你需要一个一个的手动去创建，并配置 `ssh` 公钥到对应平台，否则，后续的 `ssh` 创建过程， 会覆盖掉之前创建的。
2. `ssh-keygen` 会同时创建 `id_rsa` 和 `id_rsa.pub` 两个文件， `.pub` 是公钥， 不带后缀的是你的私钥。
3. 同时配置多个 `ssh` 时，需要您保留私钥在 `.ssh` 目录下，为保证多平台都可以使用，您需要手动修改一下 `id_rsa` 文件的名称，`.pub` 就随便了，配置完，就没用了。
4. 将密钥添加到 `ssh-agent` 中

    ```
    ssh-add ~.ssh/id_rsa
    ```
    
    ```
    # 查看 agent 中的密钥
    ssh-add -l
    # 查看 agent 中的公钥
    ssh-add -L
    # 删除 agent 中的密钥
    ssh-add -d .ssh/id_xxx.pub
    ```


## 修改配置文件

说了半天，终于到重点了

1. 在 `~/.ssh` 目录下新建一个 `config` 文件

    对，没有错，就是一个连后缀都没有的文件，这个文件用来存储您的所有平台信息，以及平台对应使用的 `ssh` 密钥。
    `touch config`

2. 添加以下配置信息

    ```
    # github
    Host github.com # 也可以是数字 ip 地址，加不加 http/https 都无所谓
    HostName github.com # 同上
    PreferredAuthentications publickey # 这里不要修改
    IdentityFile ~/.ssh/id_rsa_github # 这里的文件名修改为该平台对应的密钥
    
    # gitlab
    Host 公司 gitlab 地址
    HostName 公司 gitlab 地址
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_gitlab
    
    ...
    ```

## 测试

命令为：

```
ssh -T git@github.com
```

结果

```
Hi boywithsmalleyes! You've successfully authenticated, but GitHub does not provide shell access.
```

只需要替换后面的 `git`仓库地址, 其他版本库都可以进行测试。



## 结语

说了半天， 还是要记住一点，既然要使用 `ssh` 的方式进行版本管理，那么在 `clone` 代码时，也要使用 `ssh` 方式， 不然我说了这么半天，都是白扯。







> 文章内容有瑕疵，请给予指正批评

