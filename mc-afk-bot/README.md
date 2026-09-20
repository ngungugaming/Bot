# Minecraft Bedrock AFK Bot (Web GUI)

Node.js + Express + Socket.io + `bedrock-protocol`, with a Vietnamese-language
web control panel and a single-executable build via `pkg`.

## 1. Run in development

```bash
npm install
npm start
```

Then open **http://localhost:3000** (or the `PORT` env var if you set one).

## 2. Using the GUI

1. **Tên tài khoản** — enter the Microsoft email or Xbox Gamertag the bot
   should log in with.
2. **Tên máy chủ / IP máy chủ** — pre-filled with `CatMine` / `catmine.net`
   / port `19132`; edit if pointing at a different GeyserMC-enabled server.
3. Click **Đăng Nhập**. Since auth is `microsoft` (device code flow), a box
   will appear with a verification URL + code — open the URL in any browser,
   enter the code, and approve the sign-in. The same info is also printed to
   the terminal console.
4. Once signed in, status flips to **ON** and the bot idles (AFK) on the
   server.
5. Type in the chat box and press **Gửi** to send a message. This also opens
   a **2-minute listening window**: incoming chat is only rendered in
   "Nơi nhận tin nhắn" for those 2 minutes after your last sent message.
   After that, incoming chat is ignored until you send another message.
6. The chat buffer is capped at **2 MB**; once exceeded, the oldest lines
   are dropped automatically (both server-side memory and the DOM).

## 3. Project structure

```
mc-afk-bot/
├── server.js          # Express + Socket.io + bedrock-protocol backend
├── package.json        # dependencies + pkg build config
├── public/
│   ├── index.html       # Vietnamese-language GUI
│   └── script.js         # client-side socket.io + DOM logic
└── README.md
```

## 4. Bundling into a single executable

This project uses [`pkg`](https://github.com/vercel/pkg) to bundle Node.js,
the app code, and the `public/` assets into one standalone binary — no
Node.js install needed on the target machine.

```bash
npm install
npm run build
```

This produces, inside `dist/`:

- `afk-bot-win.exe`   (Windows x64)
- `afk-bot-linux`     (Linux x64)
- `afk-bot-macos`     (macOS x64)

(exact filenames depend on your `pkg` version's target suffixing — check the
`dist/` folder after building.)

Run the resulting binary directly, e.g. on Windows:

```
afk-bot-win.exe
```

It starts the same Express server on port 3000 (or `PORT` env var) with the
GUI embedded — no `node_modules` or source files required alongside it.

### Alternative: esbuild + Node's `--experimental-sea-config` (SEA)

If you prefer not to use `pkg` (which is community-maintained and can lag on
newer Node versions), you can bundle with `esbuild` and use Node.js's native
**Single Executable Application** feature (Node 20+):

```bash
npx esbuild server.js --bundle --platform=node --outfile=dist/bundle.js
node --experimental-sea-config sea-config.json
node -e "require('fs').copyFileSync(process.execPath, 'dist/afk-bot.exe')"
npx postject dist/afk-bot.exe NODE_SEA_BLOB dist/sea-prep.blob \
  --sentinel-fuse NODE_SEA_FUSE_fce680ab2cc467b6e072b8b5df1996b2
```

This is more manual to set up (native modules like `bedrock-protocol`'s
dependencies may need extra handling), which is why `pkg` is the default
`npm run build` path here.

## 5. Notes on `bedrock-protocol`

- `auth: 'microsoft'` delegates to `prismarine-auth`'s Authflow, which
  caches tokens locally after the first successful device-code login, so
  subsequent runs typically won't require re-authenticating in the browser.
- `version: false` lets the client auto-negotiate the Bedrock protocol
  version with the target server; pin it explicitly (e.g. `'1.21.0'`) if a
  server requires an exact match.
- Make sure the target Java server actually has **GeyserMC** (+ Floodgate,
  if you want to skip Java-account linking) running so Bedrock clients can
  join at all.
