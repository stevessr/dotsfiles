# 先得有python
- 这里以debian 12 为例
```shell
sudo apt install python3.11 -y
```
# 创建虚拟环境并激活
```bash
python -m venv venv 
source venv/bin/activate
```

or broken 
```shell
sudo mv /usr/lib/python3.13/EXTERNALLY-MANAGED /usr/lib/python3.13/EXTERNALLY-MANAGED.bk
```