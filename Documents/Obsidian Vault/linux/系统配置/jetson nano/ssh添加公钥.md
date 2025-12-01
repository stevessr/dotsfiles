# 一：在本地生成
```shell
ssh-keygen
```
# 二：上传公钥
```path
~/.ssh/id_rsa.pub
```

```shell
cd 
mkdir .ssh 
cd .ssh
touch authorized_keys 
# 上传你的公钥
cat id_ed25519.pub >> authorized_keys
```