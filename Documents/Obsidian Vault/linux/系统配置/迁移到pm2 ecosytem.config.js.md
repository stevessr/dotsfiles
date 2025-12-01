# memos

```javascript
module.exports = {
  apps: [
    {
      name: "memosp",
      script: "/mnt/sdcard/memos/memos",
      cwd: "/mnt/sdcard/memos",
      autorestart: true,
      error_file: "/mnt/sdcard/memos/logs/error.log",
      out_file: "/mnt/sdcard/memos/logs/out.log",
      env: {
        MEMOS_DRIVER: "postgres",
        MEMOS_DSN: "postgresql://memos:memos@localhost:5432/memos?sslmode=disable", // Consider enabling SSL in production
        MEMOS_MODE: "prod",
        MEMOS_PORT: "5230",
        MEMOS_DATA: "/mnt/sdcard/memos",
      },
    },
  ],
};
```

# new-api

```javascript
module.exports = {
  apps: [
    {
      name: "new-api",
      script: "/mnt/sdcard/new-api/new-api-arm64",
      cwd: "/mnt/sdcard/new-api",
      autorestart: true,
      error_file: "/mnt/sdcard/new-api/logs/error.log",
      out_file: "/mnt/sdcard/new-api/logs/out.log",
      env: {
         STREAMING_TIMEOUT:"6000",
 REDIS_CONN_STRING:"redis://localhost:6379/1",
 SQL_DSN:"postgres://newapi:newapi@localhost/newapi",
 MEMORY_CACHE_ENABLED:"true",
 PORT:"8081",
 TIKTOKEN_CACHE_DIR:"./cache/tiktoken",
      },
    },
  ],
};
```

# openwebui
```javascript
module.exports = {
  apps: [
    {
      name: "open-webui",
      script: "python3",
      arg:"/home/stevessr/.local/bin/open-webui  serve --port 8088",
      cwd: "/mnt/sdcard/open-webui ",
      autorestart: true,
      error_file: "/mnt/sdcard/open-webui/logs/error.log",
      out_file: "/mnt/sdcard/open-webui/logs/out.log",
env: {
  ENV: "prod",
  WEBUI_NAME: "steve's webui",
  CUSTOM_NAME: "steve's webui",
  WEBUI_URL: "https://any.aaca.eu.org",
  PORT: "8088",
  ENABLE_CHANNELS: "TRUE",
  DEFAULT_LOCALE: "zh_cn",
  AIOHTTP_CLIENT_TIMEOUT_OPENAI_MODEL_LIST: "10",
  DATA_DIR: "/mnt/sdcard/opwebui",
  DATABASE_URL: "postgresql://opwebui:opwebui@localhost/opwebui",
  ENABLE_WEBSOCKET_SUPPORT: "TRUE",
  WEBSOCKET_MANAGER: "REDIS",
  WEBSOCKET_REDIS_URL: "redis://localhost:6379/0",
  REDIS_URL: "redis://localhost:6379/0",
  HF_ENDPOINT: "https://hf-mirror.com"
}
    },
  ],
};
```