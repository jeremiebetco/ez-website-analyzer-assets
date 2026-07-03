# Installation

## Binary install (recommended)

### 1. Download the binary

Get the release for your platform from [GitHub Releases](https://github.com/jeremiebetco/ez-website-analyzer-assets/releases):

| Platform | File |
|----------|------|
| Linux (x86_64) | `ez-website-analyzer-linux-x86_64` |
| Windows (x86_64) | `ez-website-analyzer-windows-x86_64.exe` |
| macOS (Apple Silicon) | `ez-website-analyzer-macos-arm64` |

Put the binary in a folder of your choice.

### 2. Set API keys

Use **shell environment variables**, a `.env` file next to the binary, and/or `~/.ez-web/.env`. On first run, the binary creates `~/.ez-web/.env` from the bundled template if it does not exist.

**Resolution order** (highest priority first):

1. Shell / system environment variables
2. `.env` in the same folder as the binary (fills gaps not set in the shell)
3. `~/.ez-web/.env` (fills remaining gaps; created automatically on first run if missing)

**.env file** (either location):

```
FIRECRAWL_API_KEY=fc-your_actual_key_here
OPENAI_API_KEY=sk-your_actual_key_here
```

**Shell environment:**

```bash
export FIRECRAWL_API_KEY=fc-your_actual_key_here
export OPENAI_API_KEY=sk-your_actual_key_here
```

Shell variables take precedence over both `.env` files.

See [README Setup → Configuration & paths](README.md#configuration--paths) for the full source-vs-binary comparison table.

### 3. Run

**macOS / Linux:**

```bash
chmod +x ez-website-analyzer-macos-arm64   # use your platform file
./ez-website-analyzer-macos-arm64
```

**Windows:**

```bat
ez-website-analyzer-windows-x86_64.exe
```

Output defaults to `~/.ez-web/runs/`; create `runs/` beside the binary to write output there instead.

## Source install

Check out this repository (or a release tag). Install Python 3.12+, run `pip install -r requirements.txt`, create `.env` from `.env.example`, and use `./analyzer.sh` (or `analyzer.bat` on Windows).
