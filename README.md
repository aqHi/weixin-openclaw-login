# Weixin OpenClaw Login

A small OpenClaw skill for debugging and documenting the official WeChat personal-account login flow introduced with the Tencent Weixin/OpenClaw integration.

## What it helps with

- Install the official plugin: `@tencent-weixin/openclaw-weixin`
- Capture the **raw QR authorization URL** instead of relying on terminal block QR rendering
- Poll QR login status from Tencent's ilink endpoints
- Diagnose `openclaw-weixin` stuck at `SETUP / no token`
- Verify local token persistence and final channel readiness

## Why this skill exists

In practice, terminal-rendered QR codes often break when they are:

- forwarded through chat apps
- rendered with non-monospace fonts
- cropped in screenshots
- viewed through message relay surfaces

This skill documents the more reliable path: extract the real `qrcode_img_content` URL returned by the upstream API and let the user open it directly on a computer screen.

## Install

### From ClawHub

```bash
clawhub install weixin-openclaw-login
```

### Manual

Copy this skill folder into your OpenClaw `skills/` directory.

## Core workflow

1. Install/update the official WeChat plugin
2. Request a fresh QR code from `https://ilinkai.weixin.qq.com/ilink/bot/get_bot_qrcode?bot_type=3`
3. Extract:
   - `qrcode`
   - `qrcode_img_content`
4. Open the QR page URL on a computer
5. Scan with WeChat and confirm
6. Poll login status using the `qrcode` token
7. Verify `openclaw-weixin` becomes `OK` in `openclaw status`

## Files

- `SKILL.md` — main skill instructions
- `references/implementation-notes.md` — lower-level implementation notes and troubleshooting context

## Related links

- GitHub repo: <https://github.com/aqHi/weixin-openclaw-login>
- ClawHub package: `weixin-openclaw-login`

## Status

Initial public release based on a real successful WeChat/OpenClaw login and recovery workflow.
