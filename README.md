# Weixin OpenClaw Login

这是一个给 OpenClaw 用的小 skill，专门处理 **微信个人号接入 OpenClaw** 的官方登录流程与排障。

默认说明以 **中文优先** 为主，因为这类插件和排障场景的主要受众大概率就是中文用户。

## 这个 skill 能做什么

- 安装官方插件：`@tencent-weixin/openclaw-weixin`
- 抓取**原始二维码授权链接**，避免依赖终端字符二维码
- 轮询腾讯 ilink 接口，查询扫码登录状态
- 排查 `openclaw-weixin` 卡在 `SETUP / no token`
- 检查本地 token 是否真正落盘
- 最终确认 OpenClaw 渠道是否已经可用

## 为什么要专门做这个 skill

实际使用里，终端渲染出来的二维码经常会在这些场景翻车：

- 通过 iMessage / Messenger / Telegram 等聊天软件转发
- 字体不是严格等宽
- 截图被裁掉边缘安静区
- 经过消息中继或平台二次渲染

相比之下，直接抓取腾讯接口返回的 `qrcode_img_content` 原始页面链接，再让用户在电脑上打开、用手机微信扫码，成功率高很多。

## 安装

### 从 ClawHub 安装

```bash
clawhub install weixin-openclaw-login
```

### 手动安装

把这个仓库里的 skill 文件夹放进你的 OpenClaw `skills/` 目录即可。

## 自带脚本

### 1. 获取新的二维码授权链接

```bash
node scripts/get-login-url.js
```

输出通常包含：

- 原始二维码页面链接
- `QRCODE=...`，可用于后续查询扫码状态

### 2. 轮询扫码状态

```bash
python3 scripts/poll-login-status.py <qrcode>
```

例如：

```bash
python3 scripts/poll-login-status.py 1cf42ee545e62408992daa64b38a37d9
```

## 核心排障流程

1. 安装或更新官方微信插件
2. 请求新的二维码：
   - `https://ilinkai.weixin.qq.com/ilink/bot/get_bot_qrcode?bot_type=3`
3. 提取：
   - `qrcode`
   - `qrcode_img_content`
4. 在电脑上打开二维码页面
5. 用手机微信扫码并确认
6. 用 `qrcode` 去轮询扫码状态
7. 用 `openclaw status` 确认 `openclaw-weixin` 是否变成 `OK`

## 仓库结构

- `SKILL.md`：主 skill 说明
- `references/implementation-notes.md`：更底层的实现与排障说明
- `scripts/get-login-url.js`：获取原始二维码授权链接
- `scripts/poll-login-status.py`：轮询扫码状态

## 相关链接

- GitHub：<https://github.com/aqHi/weixin-openclaw-login>
- ClawHub：`weixin-openclaw-login`

## 当前定位

这是一个从真实排障过程里提炼出来的 skill，目标不是讲概念，而是**尽快把微信 OpenClaw 登录打通**。
