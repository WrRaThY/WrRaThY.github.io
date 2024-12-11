---
layout: post
title:  "ZSH on Windows (git bash)"
date:   2024-12-11 17:43:45 +1300
categories: bash shell git zsh windows
---

# So... is it really possible?  
Well, yeah. But it requires some work.  
The resources I used (linked at the bottom) are a bit outdated, so here is the updated version

## 1. install [msys2](https://www.msys2.org/) and follow [this](https://stackoverflow.com/a/73565205/20691743)
   2. I didn’t follow it fully - it was hard to copy those files on windows. What worked for me was just copy-pasting things that returned as errors when running the commands below

## 2. run the following:
```shell
mkdir -p /var/lib/pacman
pacman-key --init
pacman-key --populate msys2
pacman -Syu
```

## 3. then run this:
```shell
URL=https://github.com/git-for-windows/git-sdk-64/raw/main
cat /etc/package-versions.txt | while read p v; do d=/var/lib/pacman/local/$p-$v;
 mkdir -p $d; echo $d; for f in desc files install mtree; do curl -sSL --ssl-no-revoke "$URL$d/$f" -o $d/$f;
 done; done
```

## 4. And finally:
```shell
pacman -S zsh
```

You can verify that it works (just run `zsh` in your bash). And if it does...

## 5. Add this at the end of `.bashrc`
```shell
bash -c zsh
```

# Now that we have `ZSH` - we can configure it
## 1. install [nerd fonts](https://www.nerdfonts.com/) 
and configure them (quote below)
> Change Windows Terminal settings to use Nerd-Fonts
Because we want Windows Terminal to be able to render the icons in the powerlevel10k theme correctly, we need to change the Windows Terminal configuration to use the Nerd-Font we've downloaded before. Click on Settings in the Windows Terminal menu and edit the settings.json file with your favorite text editor.
> 
> Find your wsl or wsl2 profile and add the line "fontFace": "<name-of-your-font>".

## 2. update ssh
go to `~/.ssh/config` and add `AddKeysToAgent yes` to all entries

## 3. install plugins
```bash
choco install bat

git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/MichaelAquilina/zsh-you-should-use.git $ZSH_CUSTOM/plugins/you-should-use
git clone https://github.com/fdellwing/zsh-bat.git $ZSH_CUSTOM/plugins/zsh-bat
```

## 4. configure `~/.zshrc`
```shell
(...)
ZSH_THEME="powerlevel10k/powerlevel10k"
(...)
HIST_STAMPS="mm/dd/yyyy"
(...)
plugins=(git ssh-agent zsh-autosuggestions you-should-use zsh-bat)
(...)
if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi
source /c/Users/wooha/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

you can share `~/.bash_aliases` between `.bashrc` and `.zshrc`

# tips and tricks

- you may need to add `--ssl-no-revoke` in curls (in case you use something I didn't update)
- edit `.zshrc` and put this on top if you see `Command not found compdef`:

```bash
autoload -Uz compinit
compinit
```

# sources:
- [Using zsh on Windows](https://walterteng.com/using-zsh-on-windows#replace-git-bash-with-zsh)
- [videos/windows-terminal-powerlevel10k at main · ChristianLempa/videos](https://github.com/christianlempa/videos/tree/main/windows-terminal-powerlevel10k)
- [The Only 5 Zsh Plugins You Need](https://catalins.tech/zsh-plugins/)
- [Command not found compdef](https://documentation.breadnet.co.uk/kb/mac/command-not-found-compdef/)

