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

克隆项目到本地工作端（需要输入密码）

```bash
Frank-MacBook:~ Frank$ git clone git@github.com:xxx/docker.git
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

