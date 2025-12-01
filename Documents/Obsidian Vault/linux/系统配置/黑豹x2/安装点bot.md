# kirara-ai 安装
[lss233/kirara-ai: 🤖 可 DIY 的 多模态 AI 聊天机器人 | 🚀 快速接入 微信、 QQ、Telegram、等聊天平台 | 🦈支持DeepSeek、Grok、Claude、Ollama、Gemini、OpenAI | 工作流系统、网页搜索、AI画图、人设调教、虚拟女仆、语音对话 |](https://github.com/lss233/kirara-ai) 
[连接聊天平台 | Kirara AI](https://kirara-docs.app.lss233.com/guide/configuration/im.html#qq-%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0%E6%9C%BA%E5%99%A8%E4%BA%BA)
[QQ机器人管理端](https://q.qq.com/qqbot/#/developer/sandbox)
```shell
git clone https://github.com/lss233/kirara-ai.git
cd kirara-ai
uv venv 
source .venv/bin/activate.fish
uv pip install -e .
wget https://github.com/DarkSkyTeam/kirara-webui/releases/latest/download/dist.zip
unzip dist.zip -d /tmp/web_dist
mkdir web
mv /tmp/web_dist/dist/* web/
cp config.yaml.example data/
python3 -m kirara_ai
```
https://dashboard.ngrok.com/
登录并且设置反向代理
# astrbot
[Soulter/AstrBot: ✨ 易上手的多平台 LLM 聊天机器人及开发框架 ✨ 平台支持 QQ、QQ频道、Telegram、微信、企微、飞书 | MCP 服务器、OpenAI、DeepSeek、Gemini、硅基流动、月之暗面、Ollama、OneAPI、Dify 等。附带 WebUI。](https://github.com/Soulter/AstrBot)


```shell
git clone https://github.com/Soulter/AstrBot
cd AstrBot
uv venv
source .venv/bin/activate.fish
uv pip install -r requirements.txt -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
python main.py
```