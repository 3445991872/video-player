# 视频播放器（GitHub Pages 静态版）

把原来的 Node 后端播放器，改成**纯静态**页面部署到 GitHub Pages，域名走 `*.github.io`，不再依赖 `bw4.cn`。

播放页本身已经支持「静态模式」：视频地址通过 URL 参数直接传入，浏览器端用 xgplayer / hls.js 播放，**不需要任何后端**。

## 目录结构

```
github-player/
├── index.html      # 播放页（?url= 直传视频地址即可播放）
├── gen.html        # 分享链接生成器（可视化拼 ?url= 链接）
├── assets/         # xgplayer + hls.js（已本地化，离线可用）
│   ├── xgplayer.min.js
│   ├── hls-plugin.min.js
│   └── xgplayer.css
└── .nojekyll       # 关闭 Jekyll，保证 assets 正常服务
```

## 怎么播放

把视频直链（mp4 / m3u8，需公网可访问）拼到 `?url=` 后面即可：

```
https://<user>.github.io/<repo>/?url=<视频地址 URL 编码>
```

举例（Liblib 直链）：

```
https://username.github.io/video-player/?url=https%3A%2F%2Fliblibai-online.liblib.cloud%2Fimg%2F...%2Fxxx.mp4
```

`?url=` 支持三种写法：明文 URL 编码 / base64 / base64url。其它可用参数：

| 参数 | 作用 |
|---|---|
| `url` / `u` / `v` | 视频直链（必填，静态模式） |
| `title` / `t` | 标题 |
| `next` / `n` | 下一集地址（播完自动跳转） |
| `wm` / `watermark` | 水印文字（`wmt=move` 为滚动水印） |
| `btn=名称\|链接` | 底部按钮（可多个） |
| `pbtn=名称\|链接` | 播放器内悬浮按钮（可多个） |

> 不会拼链接？打开 `gen.html` 填一下自动生成。

## 发布到 GitHub Pages

1. 在 GitHub 新建一个仓库（公开），例如 `video-player`。
2. 把本目录内容推上去：
   ```bash
   cd github-player
   git init
   git add -A
   git commit -m "static video player"
   git branch -M main
   git remote add origin https://github.com/<user>/video-player.git
   git push -u origin main
   ```
3. 仓库 **Settings → Pages → Build and deployment → Source 选 "Deploy from a branch" → Branch 选 `main` / 目录 `(root)`** → Save。
4. 等一两分钟，访问 `https://<user>.github.io/video-player/` 即出现「部署自检演示视频」。

想要 `https://<user>.github.io/`（不带仓库名）的域名：把仓库命名为 `<user>.github.io`，内容放 `main` 分支根目录即可。

## 与 Liblib 上传取链联动（可选）

原来的 liblib 服务在「下发」环节会把视频建档到 `video.bw4.cn` 后端并生成 `/p/<id>` 分享链接。
改成 GitHub Pages 后，让 liblib 服务直接生成 GitHub 链接即可：

- 在 liblib 服务 `server/config.json` 增加 `deliver.ghPagesBase`（例如 `https://<user>.github.io/video-player`）。
- 改 `server/deliver.js`：下发时不再 `POST /api/admin/videos`，而是拼
  `ghPagesBase + '/?url=' + encodeURIComponent(mp4UrlCanonical) + '&title=' + encodeURIComponent(title)`，
  把该地址写进 `job.deliver.playUrl` 和二维码。

这样草料二维码扫出来就是 `github.io` 域名，完全绕开 `bw4.cn`。

## 限制

- **纯静态**：没有后台管理、没有观看统计、没有 m3u8 服务端代理重写。
  - 统计：分享链接不带 `?id=`，统计上报自动跳过（不影响播放）。
  - m3u8：走浏览器端 hls.js；若源站有严格防盗链/CORS，可能需要在源站放行，或仍保留原后端做代理。
- 视频直链必须公网可访问且允许跨域播放（mp4 一般可直接播；Liblib OSS 直链无签名、公共可读，实测可直接播）。
- 原后端（`video.bw4.cn` 的 Node 服务）可保留做 m3u8 代理/统计，也可直接停用——播放页不再依赖它。
