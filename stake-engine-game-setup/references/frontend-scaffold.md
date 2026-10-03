# Frontend Scaffold (Phaser 4 + Vite + TypeScript)

Write these files into `<GameName>/frontend/`. The code is complete and runnable; adjust the game name and `mathDir`, then implement per-game art and event handlers. The dev RGS plugin reads the math `library/` so spins are driven by real sim values.

> Phaser 4 note: keep to the standard Scene/GameObject API. Use the Phaser skills (scenes, sprites-and-images, input-keyboard-mouse-touch, tweens, text-and-bitmaptext, scale-and-responsive) when fleshing out visuals.

---

## `package.json`
```json
{
  "name": "game-frontend",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "phaser": "^4.0.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "fzstd": "^0.1.1",
    "typescript": "^5.5.0",
    "vite": "^5.4.0"
  },
  "pnpm": { "onlyBuiltDependencies": ["esbuild"] }
}
```
`fzstd` is only used by the dev RGS to read compressed books if the uncompressed `books/*.json` is missing; it never ships in the build.

## `tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "types": ["vite/client", "node"],
    "lib": ["ES2020", "DOM", "DOM.Iterable"]
  },
  "include": ["src", "vite"]
}
```

## `index.html`
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
    <title>Game</title>
    <style>html,body{margin:0;height:100%;background:#0e0f13;overflow:hidden}#game{width:100vw;height:100vh}</style>
  </head>
  <body>
    <div id="game"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

## `vite.config.ts`
```ts
import { defineConfig } from 'vite';
import { stakeMathRgs } from './vite/devRgs';

export default defineConfig({
  base: './', // CDN-relative asset paths (required for Stake Engine static hosting)
  plugins: [
    stakeMathRgs({
      // Path to the math game's library/ folder (contains publish_files/ and books/)
      mathDir: '../../math-sdk/games/YOUR_GAME/library',
      startingBalance: 1_000_000_000, // 1000.0 in 6-dp money
      currency: 'USD',
    }),
  ],
  build: { target: 'es2020', assetsInlineLimit: 0 },
});
```

---

## `vite/devRgs.ts` — dev RGS emulator (reads the math library)
```ts
import type { Plugin, Connect } from 'vite';
import { readFileSync, existsSync } from 'node:fs';
import { resolve } from 'node:path';
import { decompress } from 'fzstd';

type Mode = { name: string; cost: number; events: string; weights: string };
type Book = { id: number; events: unknown[]; payoutMultiplier: number };
type LoadedMode = { cost: number; cumWeights: number[]; ids: number[]; total: number; books: Map<number, Book> };

export interface DevRgsOptions {
  mathDir: string;
  startingBalance?: number;
  currency?: string;
  betLevels?: number[];
  base?: string; // mount path, default '/rgs'
  socialCasino?: boolean;
}

const DEFAULT_LEVELS = [
  100000, 200000, 400000, 600000, 800000, 1000000, 2000000, 4000000,
  6000000, 8000000, 10000000, 20000000, 40000000, 100000000, 1000000000,
];

export function stakeMathRgs(opts: DevRgsOptions): Plugin {
  const base = opts.base ?? '/rgs';
  const currency = opts.currency ?? 'USD';
  const betLevels = opts.betLevels ?? DEFAULT_LEVELS;
  const config = {
    minBet: betLevels[0],
    maxBet: betLevels[betLevels.length - 1],
    stepBet: 100000,
    defaultBetLevel: 1000000,
    betLevels,
    jurisdiction: { socialCasino: !!opts.socialCasino, disabledFullscreen: false, disabledTurbo: false },
  };

  let root = process.cwd();
  let modes: Map<string, LoadedMode> | null = null;
  const wallet = { balance: opts.startingBalance ?? 1_000_000_000 };
  let activeRound: any = null;

  function load(): Map<string, LoadedMode> {
    const dir = resolve(root, opts.mathDir);
    const index = JSON.parse(readFileSync(resolve(dir, 'publish_files/index.json'), 'utf8')) as { modes: Mode[] };
    const out = new Map<string, LoadedMode>();
    for (const m of index.modes) {
      // weights: id,weight,payout
      const csv = readFileSync(resolve(dir, 'publish_files', m.weights), 'utf8').trim().split('\n');
      const ids: number[] = [];
      const cumWeights: number[] = [];
      let total = 0;
      for (const line of csv) {
        const [id, w] = line.split(',');
        total += Number(w);
        ids.push(Number(id));
        cumWeights.push(total);
      }
      // books: prefer uncompressed books/books_<mode>.json, else decompress publish_files zst (jsonl)
      const stem = m.events.replace(/\.jsonl\.zst$/, '');
      const uncompressed = resolve(dir, 'books', `${stem}.json`);
      const books = new Map<number, Book>();
      if (existsSync(uncompressed)) {
        for (const b of JSON.parse(readFileSync(uncompressed, 'utf8')) as Book[]) books.set(b.id, b);
      } else {
        const raw = decompress(new Uint8Array(readFileSync(resolve(dir, 'publish_files', m.events))));
        for (const line of new TextDecoder().decode(raw).trim().split('\n')) {
          const b = JSON.parse(line) as Book;
          books.set(b.id, b);
        }
      }
      out.set(m.name.toLowerCase(), { cost: m.cost, ids, cumWeights, total, books });
    }
    return out;
  }

  function pick(mode: LoadedMode, forcedId?: number): Book {
    if (forcedId != null && mode.books.has(forcedId)) return mode.books.get(forcedId)!;
    const r = Math.random() * mode.total;
    let lo = 0, hi = mode.cumWeights.length - 1;
    while (lo < hi) { const mid = (lo + hi) >> 1; if (mode.cumWeights[mid] < r) lo = mid + 1; else hi = mid; }
    return mode.books.get(mode.ids[lo])!;
  }

  const send = (res: any, code: number, body: unknown) => {
    res.statusCode = code;
    res.setHeader('content-type', 'application/json');
    res.end(JSON.stringify(body));
  };
  const readBody = (req: any) => new Promise<any>((ok) => {
    let s = ''; req.on('data', (c: any) => (s += c)); req.on('end', () => { try { ok(JSON.parse(s || '{}')); } catch { ok({}); } });
  });

  const middleware: Connect.NextHandleFunction = async (req, res, next) => {
    if (!req.url || !req.url.startsWith(base)) return next();
    if (!modes) modes = load();
    const path = req.url.slice(base.length).split('?')[0];
    const body = req.method === 'POST' ? await readBody(req) : {};

    if (path === '/wallet/authenticate')
      return send(res, 200, { balance: { amount: wallet.balance, currency }, config, round: activeRound });
    if (path === '/wallet/balance')
      return send(res, 200, { balance: { amount: wallet.balance, currency } });
    if (path === '/wallet/play') {
      const amount = Number(body.amount);
      const mode = modes.get(String(body.mode ?? 'base').toLowerCase());
      if (!mode) return send(res, 400, { error: 'ERR_VAL', message: 'unknown mode' });
      if (!(amount >= config.minBet && amount <= config.maxBet) || amount % config.stepBet !== 0)
        return send(res, 400, { error: 'ERR_VAL', message: 'bad amount' });
      const debit = amount * mode.cost;
      if (debit > wallet.balance) return send(res, 400, { error: 'ERR_IPB', message: 'insufficient balance' });
      wallet.balance -= debit;
      const book = pick(mode, body.replayId);
      const payout = Math.floor((amount * book.payoutMultiplier) / 100);
      activeRound = { state: book.events, payoutMultiplier: book.payoutMultiplier, payout, mode: body.mode, amount, active: true };
      return send(res, 200, { balance: { amount: wallet.balance, currency }, round: activeRound });
    }
    if (path === '/wallet/end-round') {
      if (activeRound?.active) { wallet.balance += activeRound.payout; activeRound.active = false; activeRound = null; }
      return send(res, 200, { balance: { amount: wallet.balance, currency } });
    }
    if (path === '/bet/event') return send(res, 200, { event: body.event });
    return send(res, 404, { error: 'ERR_VAL' });
  };

  return {
    name: 'stake-math-rgs',
    configResolved(c) { root = c.root; },
    configureServer(server) { server.middlewares.use(middleware); },
    configurePreviewServer(server) { server.middlewares.use(middleware); },
  };
}
```

---

## `src/config.ts` — URL params (dev defaults to local RGS)
```ts
const q = new URLSearchParams(location.search);
const origin = location.origin;

export const config = {
  sessionID: q.get('sessionID') ?? 'dev-session',
  lang: q.get('lang') ?? 'en',
  device: (q.get('device') as 'mobile' | 'desktop') ?? 'desktop',
  // In dev, the math-backed RGS is mounted on this origin at /rgs.
  rgsUrl: q.get('rgs_url') ?? `${origin}/rgs`,
  currency: q.get('currency') ?? 'USD',
  // Replay support: ?replay=<mode>:<id>
  replay: q.get('replay'),
};
```

## `src/rgs/types.ts`
```ts
export type Balance = { amount: number; currency: string };
export type BetConfig = {
  minBet: number; maxBet: number; stepBet: number; defaultBetLevel: number;
  betLevels: number[]; jurisdiction: { socialCasino: boolean; [k: string]: unknown };
};
export type BookEvent = { index: number; type: string; [k: string]: unknown };
export type Round = { state: BookEvent[]; payoutMultiplier: number; payout: number; mode: string; amount: number; active: boolean };
export type AuthResponse = { balance: Balance; config: BetConfig; round: Round | null };
export type PlayResponse = { balance: Balance; round: Round };
```

## `src/rgs/money.ts` — 6-dp integer money + display
```ts
const SCALE = 1_000_000;

const META: Record<string, { symbol: string; decimals: number; after?: boolean }> = {
  USD: { symbol: '$', decimals: 2 }, EUR: { symbol: '€', decimals: 2 }, JPY: { symbol: '¥', decimals: 0 },
  MXN: { symbol: 'MX$', decimals: 2 }, BRL: { symbol: 'R$', decimals: 2 },
  XGC: { symbol: 'GC', decimals: 2, after: true }, XSC: { symbol: 'SC', decimals: 2, after: true },
};

export const toUnits = (amount: number) => amount / SCALE;

export function display(amount: number, currency: string): string {
  const m = META[currency] ?? { symbol: currency, decimals: 2, after: true };
  const s = (amount / SCALE).toFixed(m.decimals);
  return m.after ? `${s} ${m.symbol}` : `${m.symbol}${s}`;
}
```

## `src/rgs/client.ts` — same code in dev & prod
```ts
import { config } from '../config';
import type { AuthResponse, PlayResponse, Balance } from './types';

async function post<T>(path: string, body: object): Promise<T> {
  const res = await fetch(`${config.rgsUrl}${path}`, {
    method: 'POST',
    headers: { 'content-type': 'application/json' },
    body: JSON.stringify({ sessionID: config.sessionID, ...body }),
  });
  if (!res.ok) throw new Error(`RGS ${path} -> ${res.status}`);
  return res.json() as Promise<T>;
}

export const rgs = {
  authenticate: () => post<AuthResponse>('/wallet/authenticate', {}),
  balance: () => post<{ balance: Balance }>('/wallet/balance', {}),
  play: (amount: number, mode: string, replayId?: number) =>
    post<PlayResponse>('/wallet/play', { amount, mode, ...(replayId != null ? { replayId } : {}) }),
  endRound: () => post<{ balance: Balance }>('/wallet/end-round', {}),
  event: (event: string) => post<{ event: string }>('/bet/event', { event }),
};
```

---

## `src/game/events/replay.ts` — book events → sequenced animations
```ts
import type { BookEvent } from '../../rgs/types';

export type Handler = (e: BookEvent) => void | Promise<void>;

export class ReplayEngine {
  private handlers = new Map<string, Handler>();
  on(type: string, h: Handler) { this.handlers.set(type, h); return this; }

  /** Plays each event in order, awaiting async handlers so animations sequence. */
  async play(events: BookEvent[]) {
    for (const e of events) {
      const h = this.handlers.get(e.type);
      if (h) await h(e);
      else console.warn(`[replay] no handler for event type "${e.type}"`, e);
    }
  }
}
```

## `src/game/events/handlers.ts` — map your math events here (stubs)
```ts
import type Phaser from 'phaser';
import { ReplayEngine } from './replay';
import type { BookEvent } from '../../rgs/types';

const wait = (scene: Phaser.Scene, ms: number) =>
  new Promise<void>((r) => scene.time.delayedCall(ms, r));

// Register one handler per math event `type`. Replace these stubs with real animations.
export function buildReplay(scene: Phaser.Scene): ReplayEngine {
  return new ReplayEngine()
    .on('reveal', async (_e: BookEvent) => { /* spin reels / lay out board */ await wait(scene, 400); })
    .on('win', async (_e: BookEvent) => { /* highlight + count up */ await wait(scene, 300); })
    .on('runEnd', async (_e: BookEvent) => { /* settle */ await wait(scene, 100); });
}
```

## `src/game/scenes/BootScene.ts`
```ts
import Phaser from 'phaser';
import { rgs } from '../../rgs/client';
import type { AuthResponse } from '../../rgs/types';

export class BootScene extends Phaser.Scene {
  constructor() { super('Boot'); }

  preload() {
    // Load ALL art/fonts here — must come from the CDN in prod (relative paths, base './').
    // this.load.image('logo', 'assets/logo.png');
  }

  async create() {
    this.add.text(this.scale.width / 2, this.scale.height / 2, 'Loading…', { color: '#fff' }).setOrigin(0.5);
    let auth: AuthResponse;
    try { auth = await rgs.authenticate(); }
    catch (e) { this.add.text(20, 20, `RGS error: ${e}`, { color: '#f55' }); return; }
    this.scene.start('Game', auth);
  }
}
```

## `src/game/scenes/GameScene.ts` — HUD + spin loop (approval-compliant UI)
```ts
import Phaser from 'phaser';
import { rgs } from '../../rgs/client';
import type { AuthResponse, BetConfig } from '../../rgs/types';
import { display } from '../../rgs/money';
import { config as gameConfig } from '../../config';
import { buildReplay } from '../events/handlers';
import { showDisclaimer } from '../../disclaimer';

export class GameScene extends Phaser.Scene {
  private cfg!: BetConfig;
  private currency!: string;
  private balance = 0;
  private betIdx = 0;
  private mode = 'base';
  private spinning = false;
  private muted = false;
  private balanceText!: Phaser.GameObjects.Text;
  private betText!: Phaser.GameObjects.Text;
  private spinBtn!: Phaser.GameObjects.Text;

  constructor() { super('Game'); }

  create(auth: AuthResponse) {
    this.cfg = auth.config;
    this.currency = auth.balance.currency;
    this.balance = auth.balance.amount;
    this.betIdx = Math.max(0, this.cfg.betLevels.indexOf(this.cfg.defaultBetLevel));

    const w = this.scale.width, h = this.scale.height;
    this.balanceText = this.add.text(20, 20, '', { color: '#fff', fontSize: '20px' });
    this.betText = this.add.text(20, h - 40, '', { color: '#fff', fontSize: '18px' });

    // Bet adjust
    this.add.text(220, h - 40, '◀', { color: '#9cf', fontSize: '20px' }).setInteractive()
      .on('pointerup', () => this.changeBet(-1));
    this.add.text(260, h - 40, '▶', { color: '#9cf', fontSize: '20px' }).setInteractive()
      .on('pointerup', () => this.changeBet(1));

    // Spin button — spacebar is also bound below
    this.spinBtn = this.add.text(w / 2, h - 40, 'SPIN', { color: '#000', backgroundColor: '#5cf', fontSize: '22px', padding: { x: 18, y: 8 } })
      .setOrigin(0.5).setInteractive().on('pointerup', () => this.spin());

    // Sound toggle (required), info/disclaimer (required)
    this.add.text(w - 60, 20, '🔊', { fontSize: '22px' }).setInteractive()
      .on('pointerup', (_p: unknown, _x: number, _y: number, ev: any) => { this.muted = !this.muted; this.sound.mute = this.muted; (ev?.target as any); });
    this.add.text(w - 110, 20, 'ⓘ', { fontSize: '22px' }).setInteractive().on('pointerup', () => showDisclaimer());

    // Spacebar -> bet button (required)
    this.input.keyboard?.addKey(Phaser.Input.Keyboard.KeyCodes.SPACE)
      .on('down', () => this.spin());

    this.refresh();

    // Replay-url support: ?replay=<mode>:<id>
    if (gameConfig.replay) {
      const [m, id] = gameConfig.replay.split(':');
      this.mode = m || this.mode;
      this.spin(Number(id));
    }
  }

  private changeBet(dir: number) {
    if (this.spinning) return;
    this.betIdx = Phaser.Math.Clamp(this.betIdx + dir, 0, this.cfg.betLevels.length - 1);
    this.refresh();
  }

  private get bet() { return this.cfg.betLevels[this.betIdx]; }

  private refresh() {
    this.balanceText.setText(`Balance: ${display(this.balance, this.currency)}`);
    this.betText.setText(`Bet: ${display(this.bet, this.currency)}`);
  }

  private async spin(replayId?: number) {
    if (this.spinning) return;
    this.spinning = true;
    this.spinBtn.setAlpha(0.5);
    try {
      const play = await rgs.play(this.bet, this.mode.toUpperCase(), replayId);
      this.balance = play.balance.amount; // debited
      this.refresh();

      const replay = buildReplay(this);
      await replay.play(play.round.state as any);

      const end = await rgs.endRound();   // credits win
      this.balance = end.balance.amount;
      this.refresh();
      if (play.round.payout > 0)
        this.flashWin(display(play.round.payout, this.currency));
    } catch (e) {
      console.error(e);
    } finally {
      this.spinning = false;
      this.spinBtn.setAlpha(1);
    }
  }

  private flashWin(text: string) {
    const t = this.add.text(this.scale.width / 2, this.scale.height / 2, `WIN ${text}`, { color: '#ffd34d', fontSize: '40px' }).setOrigin(0.5);
    this.tweens.add({ targets: t, alpha: 0, y: '-=40', duration: 1200, onComplete: () => t.destroy() });
  }
}
```

## `src/disclaimer.ts` — required rules/info disclaimer (see stake-game-disclaimer)
```ts
const TEXT = `Malfunction voids all wins and plays. A consistent internet connection is required. \
In the event of a disconnection, reload the game to finish any uncompleted rounds. \
The expected return is calculated over many plays. The game display is not representative of any physical device \
and is for illustrative purposes only. Winnings are settled according to the amount received from the Remote Game Server \
and not from events within the web browser. TM and © 2025 Stake Engine.`;

export function showDisclaimer() {
  // Minimal DOM popup; replace with a styled in-game rules panel.
  alert(TEXT); // TODO: render in an in-game info/rules screen, not a browser alert.
}
```

## `src/main.ts` — Phaser 4 game instance
```ts
import Phaser from 'phaser';
import { BootScene } from './game/scenes/BootScene';
import { GameScene } from './game/scenes/GameScene';

new Phaser.Game({
  type: Phaser.AUTO,
  parent: 'game',
  backgroundColor: '#0e0f13',
  scale: {
    mode: Phaser.Scale.FIT,           // responsive (desktop/mobile/popout) — approval requirement
    autoCenter: Phaser.Scale.CENTER_BOTH,
    width: 1280,
    height: 720,
  },
  scene: [BootScene, GameScene],
});
```

---

## `.vscode/launch.json` — Run and Debug play button (uses the default browser)

Write this so the user can just press **F5** / the Run and Debug ▶ button: it
starts the Vite dev server and, when ready, opens the game in their **default
installed browser** (no Chrome/Edge dependency — `openExternally` uses the OS
default). Stopping the session stops the server. Put it at the **workspace root**
(`.vscode/`), and point `cwd` at the frontend folder.

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "▶ Play (default browser)",
      "type": "node-terminal",
      "request": "launch",
      "command": "pnpm dev",
      "cwd": "${workspaceFolder}/<GameName>/frontend",
      "serverReadyAction": {
        "pattern": "(https?://localhost:[0-9]+)",
        "uriFormat": "%s",
        "action": "openExternally"
      }
    }
  ]
}
```

If the frontend **is** the workspace root, use `"cwd": "${workspaceFolder}/frontend"`
(or drop `cwd` if `pnpm dev` runs from root). Optionally add a `.vscode/tasks.json`
with `frontend: install` / `frontend: build` / `math: regenerate library` shell
tasks for one-click access from *Terminal → Run Task…*.

## Run
```bash
cd <GameName>/frontend
pnpm install      # or npm install
pnpm dev          # opens Vite; rgs_url defaults to the local math-backed RGS
```
Click **SPIN** (or press space): the dev RGS weighted-picks a real book from `../../math-sdk/games/<GameName>/library`, returns its events, and the `ReplayEngine` animates them.

## Dev gotchas (these bite on first run — pre-empt them)

- **`@types/node` + `"node"` in tsconfig `types`** (already added above): `vite/devRgs.ts`
  uses Node APIs, so without them `tsc --noEmit` (the `build` step) fails.
- **pnpm 10 skips esbuild's build script** by default, then `vite build` fails. The
  `pnpm.onlyBuiltDependencies: ["esbuild"]` field above fixes it; otherwise run
  `pnpm rebuild esbuild` once.
- **Big books / dev RGS memory:** games with long event streams produce large book
  files (hundreds of MB at ~2e4 sims for deep games). The `devRgs.ts` above loads
  **every** mode eagerly on first request — for such games, make it **lazy-load per
  mode** (parse only the mode being played) and keep dev sim counts at **1e4–1e5**.
  Also ensure the math produced the uncompressed `library/books/books_<mode>.json`
  **array** (see custom-game-recipe.md) — the SDK's own uncompressed output is
  `.jsonl` lines, a different format, and Python's `.zst` frames can defeat fzstd.

## Still to implement per game
- Real art/audio in `BootScene.preload` (unique assets, CDN-loaded; no web-sdk sample assets — stake-frontend-requirements).
- A handler in `handlers.ts` for **every** event `type` your math emits.
- A styled in-game rules/paytable + disclaimer panel (replace the `alert`), showing RTP, max win, mode costs, payouts.
- Autoplay with a confirmation step and a >2× mode-switch confirmation (stake-frontend-requirements).
- Mid-spin refresh preserving the selected bet (persist `betIdx`); Stake.US word-scrub + SC/GC if targeting that jurisdiction (stake-jurisdiction-requirements).
