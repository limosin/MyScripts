# ⚡ Ultimate macOS Terminal Setup (2025)

A high-performance, aesthetically pleasing terminal setup built for speed. Optimized for macOS using Ghostty, Zsh, and Rust-based modern tools.

## 📸 Overview
- **Terminal:** [Ghostty](https://ghostty.org/) (GPU-accelerated, native macOS feel)
- **Shell:** Zsh + [Antidote](https://getantidote.github.io/) (Fastest plugin manager)
- **Prompt:** [Starship](https://starship.rs/) (Minimalist, Rust-based)
- **Font:** JetBrains Mono Nerd Font
- **Modern Tools:** `zoxide`, `eza`, `bat`, `ripgrep`, `fd`, `lazygit`

---

## 🚀 1. Core Installation

### Install Dependencies (Homebrew)
First, install the essential tools and fonts.

```bash
# Install Font
brew install --cask font-jetbrains-mono-nerd-font

# Install Terminal & Tools
brew install --cask ghostty
brew install antidote starship zoxide eza bat fzf ripgrep fd fnm lazygit
```


## Ghostty Configuration
Create or edit ~/.config/ghostty/config. This setup enables a clean "glassmorphism" look and fixes common configuration errors.

```text
# ~/.config/ghostty/config

# --- Appearance ---
theme = Catppuccin Mocha
background-opacity = 0.90
background-blur-radius = 20
window-padding-x = 10
window-padding-y = 10

# --- Font ---
font-family = "JetBrainsMono Nerd Font"
font-size = 14
# Thicken font slightly for better readability on Retina displays
font-thicken = true

# --- Performance & Behavior ---
# Ghostty defaults to GPU acceleration, so explicit rendering tweaks aren't usually needed.
macos-option-as-alt = true
cursor-style = block

```

## Zsh Configuration
Create ~/.zsh_plugins.txt for Antidote to manage.

```text
mattmc3/antidote
zsh-users/zsh-autosuggestions
zsh-users/zsh-syntax-highlighting
zsh-users/zsh-completions
zsh-users/zsh-history-substring-search
```

### The Optimized .zshrc
This configuration features Lazy Loading (for Conda/GCloud), Static Plugin Loading, and Caching for instant (<100ms) startup times.

```config
# ==========================================
# 1. PLUGIN MANAGER (Antidote)
# ==========================================
# Clone antidote if it doesn't exist (automatic bootstrap)
if [[ ! -d ${ZDOTDIR:-~}/.antidote ]]; then
  git clone --depth=1 https://github.com/mattmc3/antidote.git ${ZDOTDIR:-~}/.antidote
fi
source ${ZDOTDIR:-~}/.antidote/antidote.zsh
antidote load

# ==========================================
# 2. PROMPT & MODERN TOOLS
# ==========================================
# Initialize Starship (The Prompt)
eval "$(starship init zsh)"

# Initialize Zoxide (Smarter cd)
eval "$(zoxide init zsh)"

# ==========================================
# 3. ALIASES (Speed & Legacy Support)
# ==========================================
# Modern replacements
alias ls="eza --icons --git"
alias ll="eza --icons --git -l"
alias la="eza --icons --git -la"
alias cat="bat"
alias k="kubectl"

# Editor
export EDITOR='vim'

# ==========================================
# 4. PATHS & ENV VARIABLES
# ==========================================
# Local binaries
export PATH="$HOME/.local/bin:$HOME/bin:$PATH"

# Go / Golang
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Antigravity (Preserved from old config)
export PATH="/Users/somilsinghai/.antigravity/antigravity/bin:$PATH"

# ==========================================
# 5. SDKs & CLOUD TOOLS
# ==========================================

# Fallback: Lazy Load NVM (Only if you didn't switch to FNM)
export NVM_DIR="$HOME/.nvm"
nvm() {
   unset -f nvm
   [ -s "$(brew --prefix nvm)/nvm.sh" ] && source "$(brew --prefix nvm)/nvm.sh"
   nvm "$@"
}

# Lazy Load Google Cloud SDK
if [ -d "$HOME/Downloads/google-cloud-sdk/bin" ]; then
    export PATH=$PATH:$HOME/Downloads/google-cloud-sdk/bin
    gcloud() {
        if [ -f "$HOME/Downloads/google-cloud-sdk/path.zsh.inc" ]; then
            source "$HOME/Downloads/google-cloud-sdk/path.zsh.inc"
            source "$HOME/Downloads/google-cloud-sdk/completion.zsh.inc"
        fi
        unset -f gcloud
        gcloud "$@"
    }
fi


# Miniconda (Preserved & Optimized)
conda() {
    echo "⚡ Initializing Conda..."
    unfunction conda
    
    # Original Conda Setup Block
    __conda_setup="$('$HOME/miniconda3/bin/conda' 'shell.zsh' 'hook' 2> /dev/null)"
    if [ $? -eq 0 ]; then
        eval "$__conda_setup"
    else
        if [ -f "$HOME/miniconda3/etc/profile.d/conda.sh" ]; then
            . "$HOME/miniconda3/etc/profile.d/conda.sh"
        else
            export PATH="$HOME/miniconda3/bin:$PATH"
        fi
    fi
    unset __conda_setup
    
    # Run the command user actually typed
    conda "$@"
}

# ==========================================
# 7. KEYBINDINGS (History Search)
# ==========================================
# Bind UP and DOWN arrows to history substring search
zmodload zsh/terminfo
bindkey "$terminfo[kcuu1]" history-substring-search-up
bindkey "$terminfo[kcud1]" history-substring-search-down

# Also bind standard arrow keys (sometimes terminfo is slightly off)
bindkey '^[[A' history-substring-search-up
bindkey '^[[B' history-substring-search-down

# Bind Ctrl+P / Ctrl+N for VIM users
bindkey -M vicmd 'k' history-substring-search-up
bindkey -M vicmd 'j' history-substring-search-down



# ==========================================
# 8. TRAINING WHEELS (Nudges)
# ==========================================

# Nudge for 'grep' -> 'rg'
grep() {
    echo -e "\033[0;33m💡 Tip: Try 'rg' (ripgrep) for faster searching.\033[0m"
    command grep "$@"
}

# Nudge for 'find' -> 'fd'
find() {
    echo -e "\033[0;33m💡 Tip: Try 'fd' for simpler file finding.\033[0m"
    command find "$@"
}

# Nudge for 'cd' -> 'z'
cd() {
    echo -e "\033[0;33m💡 Tip: Try 'z directory_name' to jump instantly.\033[0m"
    builtin cd "$@"
}

```

## 🛠 Usage Guide

| Goal        | Old Command         | New Command   | Why?                                |
| ----------- | ------------------- | ------------- | ----------------------------------- |
| Change Dir  | cd projects/backend | z back        | Fuzzily jumps to directory          |
| List Files  | ls -la              | ll            | Icons, git status, colors           |
| Read File   | cat file.json       | cat file.json | Syntax highlighting (mapped to bat) |
| Search Text | grep "error" .      | rg "error"    | 10x faster, ignores .gitignore      |
| Find File   | find . -name "*.go" | fd .go        | Simple syntax, faster               |
| Git UI      | git status          | lg            | Full terminal UI for Git            |