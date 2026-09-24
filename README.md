# TRIM CLI Skill

Manage TRIM NAS files, photos, applications, Docker containers, storage, and more through an AI agent.
This repository provides the Skill documentation. The CLI is distributed through the npm package [@trimjs/trim-cli](https://www.npmjs.com/package/@trimjs/trim-cli).

## Installation

Requires Node.js 18 or later. Install the Skill:

```bash
npx skills add teiron-1604/trim-cli-skill --skill trim-cli -g
```

Install the CLI version matching this Skill:

```bash
npm install -g @trimjs/trim-cli@0.1.1
trim-cli --help
```

Alternatively, run the CLI directly without a global installation:

```bash
npx -y @trimjs/trim-cli@0.1.1 --help
```

Before first use, start the interactive OAuth login flow. Sign in and approve access in your browser, then paste the authorization code into the CLI prompt:

```bash
trim-cli --profile home --host <nas-host> --port <port> login
```

Installation does not sign you in automatically. Media APIs are outside the scope of this Skill.

## Updates

```bash
npx skills update trim-cli
```

After updating the Skill, install the matching npm version specified in its CLI setup instructions.

## License

This repository is not currently released under an open-source license. Public availability does not grant permission to modify or redistribute its contents.
