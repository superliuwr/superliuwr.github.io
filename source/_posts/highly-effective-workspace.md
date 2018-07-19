---
title: Highly Effective Workspace with Macbook
date: 2018-06-10 20:09:59
categories:
- Development Environment
tags:
- VSC
- Zsh
- Vim
---
# MacOS

* 将功能键(F1-F12)设置为标准的功能键
* 设置Trackpad（触摸板）轻触为单击
* 将Dock停靠在屏幕左边
* 全键盘控制模式
* 快速锁定屏幕

<!-- more -->

# VSC

* sync settings to Github gist
* code snippets
* shortcuts
* vim mode
    * Fix Key Repeat: `defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false`
* create case-sensitive workspace
    * `hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 100g -volname workspace ~/Documents/workspace.dmg.sparseimage`
* code snippets
    * ctrl + space
    * User Snippets by language filter

# Shell

## iTerm2
* https://www.iterm2.com/
* brew cask install iterm2
* https://hiltmon.com/blog/2013/02/13/make-iterm-2-more-mac-like/
* https://medium.com/ayuth/iterm2-zsh-oh-my-zsh-the-most-power-full-of-terminal-on-macos-bdb2823fb04c
* Reuse previous sessions window

## ZSH
* 切换默认Shell为Zsh
    * `chsh -s /bin/zsh`
* tips and tricks
    * no needs of ‘cd'
    * d

## OH MY ZSH
* Install
    * `sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
* .zshrc
    * ZSH_THEME
    * Prompt
        * `PROMPT='%{$fg_bold[red]%}➜ %{$fg_bold[green]%}%p%{$fg[cyan]%}%d %{$fg_bold[blue]%}$(git_prompt_info)%{$fg_bold[blue]%}% %{$reset_color%}>'#PROMPT='%{$fg_bold[red]%}➜ %{$fg_bold[green]%}%p %{$fg[cyan]%}%c %{$fg_bold[blue]%}$(git_prompt_info)%{$fg_bold[blue]%} % %{$reset_color%}'`
    * .zshrc
* fonts
  * [powerline fonts](https://github.com/powerline/fonts)
* Custom Plugins
  * zsh-syntax-highlighting
    * `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`
    * plugins=(... zsh-syntax-highlighting)
  * zsh-autosuggestions
    * `git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`
    * plugins=(... zsh-autosuggestions)

## Homebrew
* Install
    * `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
* brew install caskroom/cask/brew-cask
* brew cask search chrome
* brew cask install google-chrome
* brew install autojump
* fonts
    * brew tap caskroom/fonts
    * brew cask install font-fira-code
* How to upgrade formulas
    * brew upgrade terraform

## Others
* open
* pbcopy
* pbpaste
* mdfind
    * mdfind -onlyin ~/Documents essay
* screencapture
    * screencapture -c -W
    * screencapture -T 10 -P image.png
    * screencapture -s -t pdf image.pdf
* say
    * say “Time for bed, Felix”
    * ls | say
* diskutil list

# Tools

## cURL

## jq

## XCode
* xcode-select --install

## Others
* Flycut
* Spectacle
* Paragon NTFS
* Fantastical 2
* Be Focused

# Vim

# Chrome
