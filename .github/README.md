# Dotfiles

Personal configuration files managed using a bare Git repository.

## Setup Method

This uses the **bare Git repository** technique for managing dotfiles, which eliminates the need for symlinks. The repository is stored in `~/.cfg` and tracks files directly in your home directory.

## Quick Reference

Use the `config` alias instead of `git` to manage your dotfiles:

```bash
config status
config add .vimrc
config commit -m "Update vimrc"
config push
```

The config alias is defined in .zshrc:
```bash
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
```

## Multi-Account Git Setup

This dotfiles configuration supports using different Git identities for personal and work projects:

- **Personal repos** (default for all locations): Uses personal GitHub account
- **Work repos** (`~/repos/` directory): Uses work email and credentials

The setup uses Git's conditional includes to automatically apply the correct configuration based on the repository location.

### Configuration Files

- `.gitconfig` - Main Git config with conditional includes
- `.gitconfig-personal` - Personal account settings (tracked in dotfiles)
- `.gitconfig-work` - Work account settings (NOT tracked, must be configured manually)

### Work Repository Directory

All work-related repositories should be cloned into `~/repos/` to automatically use your work Git identity.

What's Tracked

- .zshrc - Zsh configuration (sanitized, no sensitive data)
- .p10k.zsh - Powerlevel10k theme configuration
- .gitconfig - Git configuration with conditional includes
- .gitconfig-personal - Personal Git account settings
- .vimrc - Vim editor configuration
- .condarc - Conda package manager configuration
- .iterm2_shell_integration.zsh - iTerm2 shell integration
- .config/gh/ - GitHub CLI configuration
- .gitignore - Ignores .cfg directory and other files

What's NOT Tracked (Sensitive)

- .gitconfig-work - Work Git credentials (must be created manually)
- .zshrc.local - Local environment variables and secrets

Sensitive Data

Sensitive data (API keys, company-specific settings, work credentials) is kept out of the tracked dotfiles:

**`.zshrc.local`** - Local environment variables and secrets:
- NOT tracked by git (add to .gitignore if needed)
- Sourced at the end of .zshrc
- Must be created manually on new machines

**`.gitconfig-work`** - Work Git credentials:
- NOT tracked by git (listed in .gitignore)
- Automatically used for repos in `~/repos/`
- Must be created manually on new machines

Example .zshrc.local:
```bash
# API Keys
export MLP_API_KEY="your-api-key"

# Company-specific settings
export GOPROXY="https://artifactory.rbx.com/api/go/go-all"
export GOPRIVATE="*github.rbx.com"

# Custom aliases
alias proxyemr='autossh -M0 -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -N -D 18080 tick'
```

Example .gitconfig-work:
```
[user]
	name = Your Name
	email = your-work-email@company.com
[credential]
	username = your-work-github-username
```

## Installing on a New Machine

Option 1: Fresh Install
```bash
# Add the alias to your current shell
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'

# Clone your dotfiles repo
git clone --bare <your-repo-url> $HOME/.cfg

# Checkout the files
config checkout

# If checkout fails due to existing files:
mkdir -p .config-backup && \
config checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | \
xargs -I{} mv {} .config-backup/{}

# Try checkout again
config checkout

# Hide untracked files
config config --local status.showUntrackedFiles no

# Create .gitconfig-work with your work credentials
nano ~/.gitconfig-work
# Add the following content:
# [user]
#     name = Your Name
#     email = your-work-email@company.com
# [credential]
#     username = your-work-github-username

# Create .zshrc.local with your sensitive data
nano ~/.zshrc.local

# Restart your shell
exec zsh
```
Option 2: Quick Install Script

Save this as a script or run directly:
```bash
git clone --bare <your-repo-url> $HOME/.cfg
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
mkdir -p .config-backup
config checkout 2>&1 | egrep "\s+\." | awk {'print $1'} | xargs -I{} mv {} .config-backup/{}
config checkout
config config --local status.showUntrackedFiles no
```
Common Commands

Adding new files
```bash
config add .bashrc
config commit -m "Add bashrc"
config push
```
Checking status
```bash
config status
```
Viewing changes
```bash
config diff
```
Pulling updates
```bash
config pull
```
Viewing history
```bash
config log
```
Advantages of This Approach

1. No symlinks - Files stay in their normal locations
2. No extra tooling - Just uses Git
3. Clean home directory - Repository is hidden in .cfg
4. Easy replication - Clone and checkout on any new machine
5. Full Git features - Branches, history, remotes all work normally

Important Notes

- The bare repository is in ~/.cfg - don't delete this directory
- Always use config command, not git, when managing dotfiles
- Untracked files are hidden by default (showUntrackedFiles=no)
- Add files explicitly - they won't show up in config status until tracked
- Keep .zshrc.local and .gitconfig-work separate and never commit them
- SSH config, credentials, and other sensitive files should NOT be tracked
- Clone all work repositories into `~/repos/` to automatically use work Git identity
- Personal projects outside `~/repos/` will use your personal Git identity

Troubleshooting

Config command not found

Add the alias to your current shell:
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'

Files not showing in status

This is normal! Only explicitly tracked files show up. To add a new file:
config add <filename>

Accidentally tracking sensitive files

Remove them:
```bash
config rm --cached <filename>
config commit -m "Remove sensitive file"
```
Want to see all files (including untracked)
```bash
config status -u
```

Check which Git identity is being used

To verify which email/username will be used in the current directory:
```bash
git config user.email
git config user.name
```

For work repos in `~/repos/`, this should show your work email. Elsewhere, it should show your personal email.

## References

- https://news.ycombinator.com/item?id=11070797
- https://www.atlassian.com/git/tutorials/dotfiles
