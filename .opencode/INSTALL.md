# Installing ChainStream Skills for OpenCode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed
- Node.js >= 18

## Installation

```bash
git clone https://github.com/chainstream-io/skills ~/.opencode/chainstream
mkdir -p ~/.agents/skills
ln -s ~/.opencode/chainstream ~/.agents/skills/chainstream
```

## Configure

```bash
npx @chainstream-io/cli login         # Create Turnkey wallet (recommended)
# or
npx @chainstream-io/cli config set --key apiKey --value <your_key>
```

## Verify

After restarting OpenCode, ask: "What ChainStream skills are available?"

## Updating

```bash
cd ~/.opencode/chainstream && git pull
```
