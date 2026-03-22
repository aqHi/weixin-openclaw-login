---
name: weixin-openclaw-login
description: Diagnose and complete WeChat/OpenClaw personal-account login via the official `@tencent-weixin/openclaw-weixin` plugin. Use when installing the plugin, generating a fresh QR login URL, checking QR status, recovering from `SETUP / no token`, or documenting how WeChat 8.0.70+ connects to OpenClaw.
---

# Weixin OpenClaw Login

Use this skill to install, debug, or document the official WeChat ↔ OpenClaw login flow.

## Quick workflow

1. Install or update the plugin:
   - `npx -y @tencent-weixin/openclaw-weixin-cli install`
2. If terminal QR rendering is inconvenient, fetch the raw QR login URL directly.
3. Ask the user to open the URL on a computer and scan it with WeChat.
4. Poll login status until `confirmed` or a `bot_token` appears.
5. Verify channel state with `openclaw status`.
6. If login succeeded upstream but OpenClaw still shows `SETUP / no token`, inspect the local state files under `~/.openclaw/openclaw-weixin/`.

## Getting the raw authorization link

The plugin source shows that login uses Tencent's ilink endpoints and returns a direct QR page URL.

Preferred command:

```bash
node - <<'NODE'
const url='https://ilinkai.weixin.qq.com/ilink/bot/get_bot_qrcode?bot_type=3';
fetch(url)
  .then(r=>r.json())
  .then(j=>{
    console.log(j.qrcode_img_content || '');
    console.log('QRCODE=' + j.qrcode);
  });
NODE
```

Expected fields:
- `qrcode_img_content`: the real QR page URL to open in a browser
- `qrcode`: the QR token used for status polling

This is usually more reliable than forwarding terminal block-art QR codes through chat channels.

## Polling login status

Use the `qrcode` token from the previous step:

```bash
python3 - <<'PY'
import urllib.request, json
qrcode = 'REPLACE_ME'
url = f'https://ilinkai.weixin.qq.com/ilink/bot/get_qrcode_status?qrcode={qrcode}'
req = urllib.request.Request(url, headers={'iLink-App-ClientVersion':'1'})
with urllib.request.urlopen(req, timeout=40) as r:
    print(json.loads(r.read().decode()))
PY
```

Useful states:
- `wait`: nobody scanned yet
- `scaned`: scanned but not fully confirmed
- `confirmed`: login accepted
- response may also include `bot_token`, `ilink_bot_id`, `baseurl`, `ilink_user_id`

If `bot_token` appears, the upstream login succeeded even if OpenClaw has not updated its own local status yet.

## Where OpenClaw stores WeChat login state

Inspect:

- `~/.openclaw/openclaw-weixin/accounts.json`
- `~/.openclaw/openclaw-weixin/accounts/<account-id>.json`
- optional sync buffer files like `*.sync.json`

A healthy account file usually contains:

```json
{
  "token": "<bot_token>",
  "savedAt": "<timestamp>",
  "baseUrl": "https://ilinkai.weixin.qq.com"
}
```

Normalized account ids typically convert `@` and `.` into `-`, for example:
- raw: `4b22f436d38f@im.bot`
- normalized: `4b22f436d38f-im-bot`

## Verification

Run:

```bash
openclaw status
```

Success looks like:
- channel `openclaw-weixin`
- state `OK`
- detail showing account count instead of `no token`

Failure looks like:
- `SETUP`
- `no token`

## When the scan succeeded but OpenClaw still shows `no token`

1. Confirm the poll result already returned `bot_token`.
2. Check whether the expected files under `~/.openclaw/openclaw-weixin/` were created.
3. If files exist, restart gateway:
   - `openclaw gateway restart`
4. Re-check `openclaw status`.

If you need more implementation detail, read `references/implementation-notes.md`.
