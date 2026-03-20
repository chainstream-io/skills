# Installing ChainStream Skills for Codex

## Prerequisites

- Git
- Node.js >= 18
- ChainStream CLI (`npx @chainstream-io/cli`)

## Installation

```bash
git clone https://github.com/chainstream-io/skills ~/.codex/chainstream
mkdir -p ~/.agents/skills
ln -s ~/.codex/chainstream ~/.agents/skills/chainstream
```

## Configure

```bash
npx @chainstream-io/cli login         # Create wallet (no email required)
# or
npx @chainstream-io/cli config set --key apiKey --value <your_key>  # API key only (read-only)
```

## Verify

```bash
ls -la ~/.agents/skills/chainstream
```

You should see two skill directories: `chainstream-data`, `chainstream-defi`.

## Updating

```bash
cd ~/.codex/chainstream && git pull
```
