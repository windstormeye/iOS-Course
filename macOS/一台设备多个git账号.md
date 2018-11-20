# 一台设备多个 git 账号
## 取消全局用户信息
```shell
# 通过以下语句查看设置的账户信息
$ git config --global user.name
$ git config --global user.email

# 取消设置的全局账户
$ git config --global --unset user.name
$ git config --global --unset user.email

# 通过以下语句设置的账户信息
$ git config --global user.name "your name"
$ git config --global user.email "your email"
```

## 重新生成新的 ssh_key
`$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_pjhubs -C "877302410@qq.com"`

## 在 github or gitlab 上添加生产密钥
```shell
$ cd ~/.ssh
# cat 出来的内容全部复制
$ cat id_rsa_pjhubs
```

## 本地识别
```shell
# 使用-K可以将私钥添加到钥匙串，不用每次开机后还要再次输入这条命令
$ ssh-add -K ~/.ssh/id_rsa_pjhubs        
# 查看已添加的内容
$ ssh-add -l
```

## 创建 config 文件，使不同的项目可用不同的信息进行处理
```shell
$ cd ~/.ssh/
# 没有config文件时，用以下命令生成
$ touch config
```

config 文件中键入以下内容：

```shell
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_pjhubs

Host git.xxx.com
HostName git.xxx.com
User git
IdentityFile ~/.ssh/id_rsa

# 如果还有继续添加
```

## 验证
```shell
$ ssh -vT git@github.com
$ ssh -vT git@git.xxx.com

# 出现 Hi，your name 等信息就稳妥了
```



