# deepseek-super-agent

A proactive coding agent for one GitHub repository, packaged as a bundle for [DeepSeek Harness](https://www.deepseek.com/en/harness/). It listens to GitHub and, optionally, Feishu. When an allowed person asks for work, it opens a Ticket, works on it in its own Session and directory, and answers in the original conversation, for example with a pull request. A web panel shows what needs a person.

## Requirements

- Node.js 22.19+ or 24+
- pnpm
- git 2.32+
- tmux and Python 3.8+
- GitHub CLI (`gh`) 2.48+
- A model provider configured in DeepSeek Harness
- Optional, for Feishu: `lark-cli` and a Feishu or Lark app with the bot capability

The agent also needs a GitHub user account of its own: it comments, pushes and opens pull requests as that account, never as yours. See [SETUP.md](SETUP.md) for the account, the repository and the Policy.

## Install

Run these commands once, on the machine that hosts the agent.

1. Install DeepSeek Harness. The bundle requires a version inside the `engines.dsh` range of its `package.json`; this release is tested with `0.2.1-rc.3`:

   ```sh
   npm install -g @deepseek-ai/dsh@0.2.1-rc.3
   ```

2. Clone this repository and build the bundle:

   ```sh
   git clone https://github.com/gaoguosheng/deepseek-super-agent.git
   cd deepseek-super-agent
   pnpm install
   pnpm build
   ```

3. Create a dedicated profile from the DeepSeek Harness Web template. Keep it separate from the profile you work in, so restarts and upgrades do not close your own Sessions. Run this only on a first install:

   ```sh
   dsh superagent --from-default-profile web --dump-config > /dev/null
   ```

4. Install the bundle into that profile:

   ```sh
   dsh plugin --profile superagent add "link:$PWD"
   ```

5. Write the machine configuration to `${DSH_HOME:-$HOME/.dsh}/profiles/superagent/cordis.patch.yml`, replacing the `stateDir` placeholder with an absolute path:

   ```yaml
   - id: webserver
     config:
       host: 127.0.0.1
       port: 3082
   - id: dsh-superagent
     config:
       stateDir: /absolute/path/to/.dsh/state/superagent
   # Feishu is optional: uncomment to enable its Source, then set the bot's lark-cli on the Feishu Source.
   # - id: dsh-superagent-feishu
   #   disabled: false
   ```

   `stateDir` holds the database, the lock, the command shims, the agent's git configuration and the Ticket directories. Keep it stable, and back it up before an upgrade.

6. Start the profile:

   ```sh
   dsh superagent
   ```

   The panel is at <http://127.0.0.1:3082>. Select **Superagent** in the sidebar.

## Configure

On first start the panel reports the setup that is still missing. Point the GitHub Source at the repository and the agent's `gh` wrapper, then save the allowed logins in Policy and the executor settings in Execution. For the accounts, the optional Feishu bot, the prompts and the health check, read [SETUP.md](SETUP.md) and [docs/006-deployment.md](docs/006-deployment.md).

## Upgrade

Back up `stateDir/orchestrator.sqlite`, then:

```sh
git pull
pnpm install
pnpm build
```

Restart the profile. A database migrated by a newer version cannot be opened by an older one, so keep the backup.

## Development

`pnpm check` runs the typecheck, the tests and the build.
