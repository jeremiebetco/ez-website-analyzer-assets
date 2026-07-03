# Installation

## 1. Download the binary

Get the release for your platform from [GitHub Releases](https://github.com/jeremiebetco/ez-website-analyzer-assets/releases):

| Platform | File |
|----------|------|
| Linux (x86_64) | `ez-website-analyzer-linux-x86_64` |
| Windows (x86_64) | `ez-website-analyzer-windows-x86_64.exe` |
| macOS (Apple Silicon) | `ez-website-analyzer-macos-arm64` |

Put the binary in a folder of your choice.

## 2. Set API keys

Use **either** a `.env` file next to the binary **or** shell environment variables.

**.env file** (create in the same folder as the binary):

```
FIRECRAWL_API_KEY=fc-your_actual_key_here
OPENAI_API_KEY=sk-your_actual_key_here
```

**Shell environment:**

```bash
export FIRECRAWL_API_KEY=fc-your_actual_key_here
export OPENAI_API_KEY=sk-your_actual_key_here
```

Shell variables take precedence over `.env`.

## 3. Run

**macOS / Linux:**

```bash
chmod +x ez-website-analyzer-macos-arm64   # use your platform file
./ez-website-analyzer-macos-arm64
```

**Windows:**

```bat
ez-website-analyzer-windows-x86_64.exe
```

Output is written to a `runs/` folder next to the binary.
