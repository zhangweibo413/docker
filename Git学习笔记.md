# Git学习笔记

## GitHub使用技巧

### GitHub设置SSH Key

工作端电脑设置公私钥

```bash
Frank-MacBook:~ Frank$ ssh-keygen -t ed25519 -C "zhangweibo@gmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/Frank/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/Frank/.ssh/id_ed25519.
Your public key has been saved in /Users/Frank/.ssh/id_ed25519.pub.
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOQEswISzw5TKWtxX7yzIT/WmMjbU7IN3whyzaYULprj zhangweibo@gmail.com
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

