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

默认获得main分支，同时自动将**origin**设置为远程仓库的标识符

假如GitHub远程仓库有feature-A分支，获取远程的feature-A分支命令为

```bash
Frank-MacBook:docker Frank$ git pull origin feature-A
```

提交修改

git add 向暂存区中添加文件

git commit 保存仓库的历史记录，-m参数 记述详细提交信息

git push 这步以后github才会被更新

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

* main
```

git checkout -b	创建，切换分支

```bash
Frank-MacBook:docker Frank$ git checkout -b feature-A
Switched to a new branch 'feature-A'
```

git push	分支发布到GitHub

```bash
Frank-MacBook:docker Frank$ git push --set-upstream origin feature-A
Enter passphrase for key '/Users/Frank/.ssh/id_ed25519':
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 456 bytes | 456.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
remote:
remote: Create a pull request for 'feature-A' on GitHub by visiting:
remote:      https://github.com/xxx/docker/pull/new/feature-A
remote:
To github.com:xxx/docker.git

 * [new branch]      feature-A -> feature-A
   Branch 'feature-A' set up to track remote branch 'feature-A' from 'origin'.
```

后面push就直接git push即可

或者在GitHub网站创建feature-B分支，在客户工作端切换到feature-B，获取最新的仓库分支

```bash
Frank-MacBook:docker Frank$ git checkout feature-B
Branch 'feature-B' set up to track remote branch 'feature-B' from 'origin'.
Switched to a new branch 'feature-B'
Frank-MacBook:docker Frank$ git branch
  feature-A

* feature-B
  main
  Frank-MacBook:docker Frank$ git pull
  Enter passphrase for key '/Users/Frank/.ssh/id_ed25519':
  Already up to date.
```

在分支B新创建一个文件，发布后，可以看到分支AB内容是不一样，如果合并到main主分支，main主分支能看到AB所有的更新内容

```bash
Frank-MacBook:docker Frank$ git add Docker+k8s.md
Frank-MacBook:docker Frank$ git commit -m "add new Docker+k8s.md"
[feature-B e161652] add new Docker+k8s.md
```

分支获取到主干最新的源代码,先切换到分支B，然后把从主干合并到分支B

```bash
Frank-MacBook:docker Frank$ git checkout feature-B
Switched to branch 'feature-B'
Your branch is up to date with 'origin/feature-B'.
Frank-MacBook:docker Frank$ git merge main
Updating e161652..41a2490
Fast-forward
 "Git\345\255\246\344\271\240\347\254\224\350\256\260.md" | 26 ++++++++++++++++++++++++++
 README.md                                                |  4 ++++
 2 files changed, 30 insertions(+)
```

在本地删除分支B

```bash
Frank-MacBook:docker Frank$ git branch -D feature-B
Deleted branch feature-B (was 41a2490).
```

### 接收Pull Request

流程

开发者A先Fork一份需要修改的源代码的仓库

在开发机器Clone一份上一步Fork的远程仓库到开发机器本地仓库

可以在本地直接修改源代码，然后Push，或者创建分支修改，然后Push到远程再进行仓库合并

最后向主分支提交pull Request，要求代码合并

还有一种情况，不Fork直接从分支发送Pull Request，比较适合比较熟悉的企业内部开发团队

远程仓库的概念需要学习
今天终于搞清楚本地仓库和远程仓库命令操作的区别

更新测试

