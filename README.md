# STTlink 订阅转换

基于 [CareyWang/sub-web](https://github.com/CareyWang/sub-web) 定制的订阅转换前端。

## 主要配置

项目在构建时读取根目录 `.env`：

```env
# 定制订阅链接使用的 subconverter 域名（不要在末尾添加 /sub）
VITE_SUBCONVERTER_DEFAULT_BACKEND="https://api.appfile.top"

# 保持使用 suosuo.de 官方短链
VITE_MYURLS_API="https://suosuo.de/short"
```

当前默认使用 `https://api.appfile.top` 作为 subconverter 域名。修改该配置后需要重新构建。生成的定制订阅地址格式为：

```text
https://api.appfile.top/sub?target=clash&url=...
```

短链仍由 `https://suosuo.de/short` 生成。

## 本地构建

需要 Node.js 24 和 Yarn：

```bash
yarn install --frozen-lockfile
yarn lint
yarn build
```

## Docker

```bash
docker build -t sttlink-sub-web:latest .
docker run -d --name sttlink-sub-web --restart=always -p 127.0.0.1:8090:80 sttlink-sub-web:latest
```

## 上游与许可证

本项目是 `CareyWang/sub-web` 的修改版本，保留上游项目的 MIT License 和原始版权声明。完整许可内容见 [LICENSE](LICENSE)。

- 上游项目：https://github.com/CareyWang/sub-web
- subconverter：https://github.com/tindy2013/subconverter
- 官方短链：https://suosuo.de

对本项目的使用、修改和再发布必须继续遵守 MIT License，保留许可证文本及原始版权声明。
