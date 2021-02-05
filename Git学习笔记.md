# Git学习笔记

## GitHub使用技巧

### GitHub设置SSH Key

工作端电脑设置公私钥

```bash
Frank-MacBook:~ Frank$ ssh-keygen -t ed25519 -C "xxx@gmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/Frank/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/Frank/.ssh/id_ed25519.
Your public key has been saved in /Users/Frank/.ssh/id_ed25519.pub.
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOQEswISzw5TKWtxX7yzIT/WmMjbU7IN3whyzaYULprj xxx@gmail.com
```

 在GitHub设置公钥（复制~/.ssh/id_ed25519.pub里的内容）

![github-ssh](./images/github-ssh.png)

验证SSH Key

```bash
Frank-MacBook:.ssh Frank$ ssh -T git@github.com
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,13.229.188.59' (RSA) to the list of known hosts.
Enter passphrase for key '/Users/Frank/.ssh/id_ed25519':
Hi zhangweibo413! You've successfully authenticated, but GitHub does not provide shell access.
```

### 初始化

```bash
Frank-MacBook:docker Frank$ git config --global user.email "xxx@gmail.com"
Frank-MacBook:docker Frank$ git config --global user.name "xxx"
```

### Git基本操作命令

GitHub**远程仓库**克隆项目到本地工作端（需要输入密码）

```bash
Frank-MacBook:~ Frank$ git clone git@github.com:xxx/docker.git
```

**默认获得main分支**，同时自动将**origin**设置为远程仓库的标识符

假如GitHub远程仓库有feature-A分支，获取远程的feature-A分支命令为

```bash
Frank-MacBook:docker Frank$ git checkout -b feature-A origin/feature-A
```

在本地仓库修改文件后，提交修改

git add 向暂存区中添加文件

git commit 保存仓库的历史记录，-m参数 记述详细提交信息

git push 这步以后github远程仓库才会被更新

```bash
Frank-MacBook:docker Frank$ git add Git学习笔记.md
Frank-MacBook:docker Frank$ git commit -m "edit git学习笔记"
[main 169e0a3] edit git学习笔记
 1 file changed, 6 insertions(+)
Frank-MacBook:docker Frank$ git push
Enter passphrase for key '/Users/Frank/.ssh/id_ed25519':
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 518 bytes | 518.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:xxx/docker.git
   169e0a3..859f2f9  main -> main
```

查看提交历史

```bash
Frank-MacBook:docker Frank$ git log
commit 859f2f91979997c794e659c4a480eed2096fe0b6 (HEAD -> main, origin/main, origin/HEAD)
Author: xxxx <xxx@gmail.com>
Date:   Mon Feb 1 01:23:56 2021 +0800

    edit git study note

commit 169e0a385d6ecdf877a4a432f70bce736499e0f3
Author: xxx <xxx@gmail.com>
Date:   Mon Feb 1 01:10:19 2021 +0800

    edit git学习笔记
```

### 分支的操作

git branch	显示分支一览表

```bash
Frank-MacBook:docker Frank$ git branch
* feature-A
  main
```

git checkout 切换分支

```bash
Frank-MacBook:docker Frank$ git checkout feature-A
Switched to a new branch 'feature-A'
```

如果本地仓库没有分支，第一次获取远程仓库的分支的时候，需要加-b参数

当远程仓库分支feature-A的内容发生变化，本地仓库需要更新时候，用以下命令

```bash
acBook:docker Frank$ git pull origin feature-A
  Enter passphrase for key '/Users/Frank/.ssh/id_ed25519':
  Already up to date.
```

### 接收Pull Request

上面例子中，本地仓库不管哪个分支进行修改后，最终push到GitHub远程仓库，在GitHub网站都会收到pull request要求，你可以在网站进行审核，并且最终合并到main主分支。

Push到GitHub远程仓库后，页面会显示收到pull request要求

![](./images/github-ssh-1.png)

然后就可以进行审核并后续的合并请求操作

![](./images/github-ssh-2.png)

![](./images/github-ssh-3.png)

合并成功后，如果分支修改项目结束，可直接删除分支，如果没结束，关闭页面即可

![](./images/github-ssh-4.png)

> 备注：社会开发者流程
>
> 开发者A先Fork一份需要修改的源代码的仓库
>
> 在开发机器Clone一份上一步Fork的远程仓库到开发机器本地仓库
>
> 可以在本地直接修改源代码，然后Push，或者创建分支修改，然后Push到远程再进行仓库合并
>
> 最后向主分支提交pull Request，要求代码合并
>
> 还有一种情况，不Fork直接从分支发送Pull Request，这个比较适合比较熟悉的企业内部开发团队
>
测试一下本地获取远程仓库的更新

## 使用GitHub开发流程

### 团队使用GitHub注意事项

1. 一切从简
2. 项目管理工具与GitHub的区别（项目）
3. 不Fork仓库的方法

### Fork仓库的方法

1. 在GitHub上进行Fork
2. 在1的仓库clone至本地开发环境
3. 在本地环境中创建特性分支
4. 对特性分支进行代码修改丙进行提交
5. 将特性分支push到1的仓库中
6. 在GitHub上对Fork来源仓库发送Pull Request

在公司团队里，由于每天都见面，通过Fork仓库的流程过于复杂，可以采用更加简单的方式

### GItHuB Flow流程

整个开发流程如下

1. 令master分支市场保持可以部署的状态
2. 进行性的作业时要从master分支创建新分支，新分支名称要具有描述性
3. 在2新建的本地仓库分支中进行提交
4. 在GitHub端仓库创建同名分支，定期push
5. 需要帮助或反馈创建Pull Request，以Pull Request进行交流
6. 让其他开发者进行审查，确认作业完成后与master分支合并
7. 与master分支合并后立即部署

这个流程的说明

- 随时部署，没有发布的概念
- 进行新的作业时要从master分支创建新分支，不管是添加新功能还是bug修复，分支名字的描述性让其他开发者对正在实施的任务一目了然。
- 在创建的分支进行提交，每次Pull Request提交最好有测试代码，方便自动化测试同步更新测试代码
- 尽量缩短Pull Request的时间，不要等到要与master合并再提Pull Request
- 合并到master分支后通过所有自动化测试，需要立刻进行部署





