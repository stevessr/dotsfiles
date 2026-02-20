# 确认服务是否存在

这里以ollama为例

```shell
ls /etc/systemd/system/ | grep ollama
```

其他服务时只需替换ollama

# 修改文件

使用你的文件管理器
这里以vim为例
```shell
sudo /etc/systemd/system/ollama.service
```

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"


[Install]
WantedBy=default.target
```

这里以一些变量为例，添加后成为

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
OLLAMA_HOST="0.0.0.0"

[Install]
WantedBy=default.target
```

# 重新加载
```shell
sudo systemctl daemon-reload
```
# 重启服务
```shell
sudo systemctl restart ollama.service
```