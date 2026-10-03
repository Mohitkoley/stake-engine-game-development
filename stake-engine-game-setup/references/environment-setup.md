# Environment Setup (from an empty folder)

This is **Part 0** — run it when the working folder contains nothing but `.claude/skills/`. It makes the repo self-sufficient: the math engine and the toolchains the rest of this skill assumes. Nothing here depends on anything pre-existing in the folder.

## Prerequisites (toolchains)

- **Python 3.11+** (for the math-sdk).
- **Node 18.18+** (current LTS / Node 24 work fine with Phaser 4 + Vite 5) and **pnpm 10.5.0** (for the frontend). Install via nvm if absent:
  ```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
  . "$HOME/.nvm/nvm.sh"
  nvm install 18.18.0 && nvm use 18.18.0
  npm install -g pnpm@10.5.0
  ```

### Windows PowerShell

Install Git, Node.js 18.18 or newer, and Python 3.11 or newer. Windows does not use `nvm-sh`; install Node.js from the official installer or use a Windows Node version manager. Then run:

```powershell
npm install --global pnpm@10.5.0
```

## 1. Get the Stake Engine math-sdk

Clone it into the repo (the frontend's dev RGS reads the game's `library/` from here):
```bash
git clone https://github.com/StakeEngine/math-sdk.git
```

PowerShell:

```powershell
git clone https://github.com/StakeEngine/math-sdk.git
```

## 2. Bootstrap the Python env

Standard path (most machines):
```bash
cd math-sdk
python3 -m venv env
env/bin/python -m pip install -r requirements.txt
env/bin/python -m pip install -e .   # makes top-level src/, utils/, optimization_program/ importable
```

Windows PowerShell:

```powershell
Set-Location math-sdk
py -3 -m venv env
.\env\Scripts\python.exe -m pip install -r requirements.txt
.\env\Scripts\python.exe -m pip install -e .
```

**Fallback if `python3 -m venv env` fails with "ensurepip is not available"** (system Python with no ensurepip/pip and no sudo — e.g. some Debian/Ubuntu boxes under PEP 668):
```bash
cd math-sdk
python3 -m venv env --without-pip
curl -sS https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py
env/bin/python /tmp/get-pip.py            # PEP 668 doesn't apply inside the venv
env/bin/python -m pip install -r requirements.txt
env/bin/python -m pip install -e .
```

> The `pip install -e .` step is mandatory — without it every game's `run.py` dies with `ModuleNotFoundError: No module named 'src'`.

## 3. (Optional) Rust optimizer

The SDK ships the official optimizer (`optimization_program/`). If you intend to run the standard optimizer rather than a custom weighter, ensure a Rust toolchain is present (`rustup`); the SDK build compiles it on first optimization run. For emergent heavy-tail games where the Rust optimizer is slow, see the custom-weighter note in [math-scaffold.md](math-scaffold.md).

## Result

After this, the folder has:
```
<repo>/
  .claude/skills/          # these skills
  math-sdk/                # cloned engine, env/ ready, editable-installed
```
You can now create the game's math package (Part 1, see [math-scaffold.md](math-scaffold.md)) and the frontend (Part 2, see [frontend-scaffold.md](frontend-scaffold.md)).

On Windows, activate the environment when interactive Python commands are useful:

```powershell
.\env\Scripts\Activate.ps1
```

If PowerShell blocks local activation scripts, either run the commands through `env\Scripts\python.exe` directly or, for the current user, use:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```
