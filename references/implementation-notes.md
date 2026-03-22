# Implementation Notes

## Why the raw QR URL can be extracted

Inside the official plugin source:

- `src/auth/login-qr.ts`
  - calls `GET https://ilinkai.weixin.qq.com/ilink/bot/get_bot_qrcode?bot_type=3`
  - receives JSON with:
    - `qrcode`
    - `qrcode_img_content`
- `src/channel.ts`
  - tries to render the QR with `qrcode-terminal`
  - falls back to printing `二维码链接: ${startResult.qrcodeUrl}` if terminal QR rendering is unavailable

So the terminal QR is only a presentation layer. The real login artifact is the URL in `qrcode_img_content`.

## Why this matters

Terminal QR blocks often break when:
- relayed over iMessage, Messenger, Telegram, etc.
- fonts are not truly monospace
- screenshots crop the quiet zone

Using the raw `qrcode_img_content` URL avoids all of that.

## Poll endpoint

The plugin also uses:

`GET https://ilinkai.weixin.qq.com/ilink/bot/get_qrcode_status?qrcode=<token>`

Typical successful payload includes:

```json
{
  "baseurl": "https://ilinkai.weixin.qq.com",
  "bot_token": "<account>@im.bot:<secret>",
  "ilink_bot_id": "...",
  "ilink_user_id": "...",
  "status": "confirmed"
}
```

In practice, seeing `bot_token` is enough to know Tencent side has accepted the login.

## Local state layout

The official plugin code resolves state dir to:

- `OPENCLAW_STATE_DIR`, else
- `CLAWDBOT_STATE_DIR`, else
- `~/.openclaw`

Then it writes WeChat state under:

- `~/.openclaw/openclaw-weixin/accounts.json`
- `~/.openclaw/openclaw-weixin/accounts/*.json`

## Good operator checklist

- Prefer raw QR URLs over forwarding text QR art.
- Prefer checking poll responses before blaming the user scan.
- Verify local token persistence separately from upstream confirmation.
- Use `openclaw status` as the final truth for channel readiness.
