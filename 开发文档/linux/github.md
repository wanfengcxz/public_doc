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

