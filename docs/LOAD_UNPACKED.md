# Load this fork unpacked (Turbo: speeds above 16x)

This fork lets Global Speed go past Chrome's 16x `playbackRate` cap (up to **100x**).
Chrome rejects any rate above 16, so above 16x the extension plays at 16x and adds
periodic forward seeks to make up the difference.

## 1. Build

Requires Node.js and npm.

```bash
git clone -b turbo-speed-over-16x https://github.com/xplodman/globalSpeed.git
cd globalSpeed
npm install
NODE_ENV=development node tools/viteBuild.js
```

> Do **not** use `npm run dev`. It also calls `tools/localeTools`, which is gitignored
> upstream and not in the repo, so it fails at the last step. The Vite build above is
> the part that matters.

## 2. Add the locale file

The manifest uses `__MSG_appName__`, so Chrome refuses to load the build without
`_locales/en/messages.json`. Generate it from the repo's English strings:

```bash
node -e "const fs=require('fs');const d=JSON.parse(fs.readFileSync('static/locales/en.json','utf8'));fs.mkdirSync('build/unpacked/_locales/en',{recursive:true});fs.writeFileSync('build/unpacked/_locales/en/messages.json',JSON.stringify({appName:{message:'Global Speed (Turbo >16x)'},appDesc:{message:d[':appDesc']}},null,1))"
```

Re-run this after every build (the build output is wiped each time).

## 3. Load it in Chrome

1. **Disable or remove the Chrome Web Store version of Global Speed first.** Two copies
   will fight over the same videos.
2. Open `chrome://extensions`.
3. Turn on **Developer mode** (top right).
4. Click **Load unpacked** and choose the `build/unpacked` folder.
5. Pin the extension if you like. It appears as **Global Speed (Turbo >16x)**.

Reload any tab that was already open so the content scripts attach.

## 4. Use it

Speed inputs, sliders, presets, rules and keybind limits now accept values up to 100x.
Above 16x the video or audio plays at a native 16x with forward seeks added on top.

## Limitations

- **Needs seekable media.** Streams that can't seek (live streams, servers without HTTP
  Range support) won't speed up past 16x.
- **Buffering.** Far above 16x the player has to fetch ahead; slow connections will stall.
- **Choppy.** Seeking skips frames, and audio becomes a stutter. This is meant for
  scanning long media, not listening.
- **Effective speed runs a little under target** (measured about 2% to 8% low at 32x to
  100x) because each seek takes time.
- **Not tested:** ghost mode with the seek loop, and site-specific handling (YouTube,
  Netflix, Douyin) above 16x.

## Updating

```bash
git pull
npm install
NODE_ENV=development node tools/viteBuild.js
# re-run the locale command from step 2
```

Then click the reload icon on the extension card in `chrome://extensions`.
