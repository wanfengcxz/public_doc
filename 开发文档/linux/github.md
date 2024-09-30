# 基本命令

```bash
# 在某个分支上向后移动一个 commit
git reset --hard HEAD~1 # --hard 会丢弃当前工作目录中未提交的更改。
git reset --soft HEAD~1	

# 将错误提交的代码恢复到未提交的状态
# 如果没有推送到远程仓库
git reset HEAD~
```



# 配置SSH密钥连接

```bash
ssh-keygen -t ed25519 -C "<your github email>"
# ed25519是目前最安全、加解密速度最快的key类型。
# 但有些旧版本的SSH还不支持ed25519算法，这时候可以用rsa算法。
# 因此，有ed25519就用ed25519，没有就用rsa。
ssh-keygen -t rsa -b 4096 -C "your github email"

```

会得到两个文件

```bash
/root/.ssh/id_ed25519
/root/.ssh/id_ed25519.pub
```

将公钥复制到https://github.com/settings/keys即可

由于配置的是ssh配置，那么在提交代码时，源仓库必须是ssh仓库。如果源仓库是https，需要手动修改。
```bash
git remote set-url origin git@github.com:DeepLink-org/deeplink.framework.dev.git
```