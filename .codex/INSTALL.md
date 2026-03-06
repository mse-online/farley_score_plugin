# Installing Farley Score for Codex

Enable Farley score skills in Codex via native skill discovery. Just clone and symlink.

## Prerequisites

- Git

## Installation

1. **Clone the Farley score repository:**
   ```bash
   git clone https://github.com/mse-online/farley_score_plugin.git ~/.codex/farley_score_plugin
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/farley_score_plugin/skills ~/.agents/skills/farley_score_plugin
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\farley_score_plugin" "$env:USERPROFILE\.codex\farley_score_plugin\skills"
   ```

3. **Restart Codex** (quit and relaunch the CLI) to discover the skills.

## Verify

```bash
ls -la ~/.agents/skills/farley_score_plugin
```

You should see a symlink (or junction on Windows) pointing to your Farley score skills directory.

## Updating

```bash
cd ~/.codex/farley_score_plugin && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/farley_score_plugin
```

Optionally delete the clone: `rm -rf ~/.codex/farley_score_plugin`.
