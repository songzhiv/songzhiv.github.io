# 

# 配置

## 重回Ubuntu

### 换源

```bash
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo apt update
```

### 代理

[https://zhuanlan.zhihu.com/p/108927713](https://zhuanlan.zhihu.com/p/108927713)

## 软件安装与配置

```bash
终端
sudo apt install zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions

neovim
sudo apt install -y neovim
sudo apt autoremove vim
sudo apt autoremove nano
sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

mkdir ~/.config/nvim/
nvim ~/.config/nvim/init.vim
curl -sL install-node.now.sh/lts | bash
sudo apt-get install yarn

:CocInstall coc-cmake  coc-git coc-highlight  coc-json  coc-python coc-sh  coc-snippets coc-syntax

software
sudo apt upgrade
sudo apt install python2.7 
sudo apt install python3-pip
pip3 install -i https://mirrors.ustc.edu.cn/pypi/web/simple pip -U
pip3 config set global.index-url https://mirrors.ustc.edu.cn/pypi/web/simple

git挂代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:10808
git config --global http.https://github.com.proxy https://127.0.0.1:10809
git config --global https.https://github.com.proxy https://127.0.0.1:10809
git config --global --unset http.proxy
git config --global --unset https.proxy

export http_proxy="http://localhost:port"
export https_proxy="http://localhost:port"

alias setproxy="export ALL_PROXY=socks5://127.0.0.1:10808" 
alias unsetproxy="unset ALL_PROXY"

docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt update 
sudo apt install -y docker-ce
用管理员打开windows terminal
sudo service docker start
docker ps
sudo vi /etc/docker/.json
添加=============================
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
pip3 install docker-compose

sudo git clone --depth=1  https://github.com/maurosoria/dirsearch.git /opt/dirsearch
    sudo ln -sf /opt/dirsearch/dirsearch.py  /usr/local/bin/dirsearch

sudo git clone --depth=1 https://github.com/offensive-security/exploitdb.git /opt/exploitdb
sudo ln -sf /opt/exploitdb/searchsploit /usr/local/bin/searchsploit

sudo git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall && \
  chmod 755 msfinstall && \
  ./msfinstall
  
sudo apt install nmap
sudo apt install hydra
sudo git clone --depth=1 https://github.com/vulhub/vulhub.git
```

### init.vim

```bash
call plug#begin('~/.config/vim_plug/plugged')
    Plug 'sainnhe/sonokai'
    Plug 'vim-python/python-syntax'
    Plug 'vim-airline/vim-airline'
    Plug 'vim-airline/vim-airline-themes'
    Plug 'luochen1990/rainbow'
    Plug 'preservim/nerdtree'
    Plug 'neoclide/coc.nvim', {'branch': 'release'}
call plug#end()

filetype plugin on
filetype indent on
set ambiwidth=double
set showmatch " 高亮匹配括号

set ignorecase
set clipboard+=unnamed
set softtabstop=4
set shiftwidth=4
set nobackup
set termguicolors
set nu "设置显示行号
syntax on "语法检测
set guicursor=i:block-iCursor-blinkon0,v:block-vCursor
set autoindent
set t_Co=256
set mouse=a
set tabstop=4
set history=1000
set encoding=utf-8
let g:rainbow_active = 1
map <Tab> :NERDTreeToggle<CR>
nmap <F12> :SCCompileRun<cr>
nnoremap <F9> :echo system('python2 "' . expand('%') . '"')<cr>
nnoremap <F10> :echo system('python3 "' . expand('%') . '"')<cr>
```

### .zshrc

```bash
# If you come from bash you might have to change your $PATH.
# export PATH=$HOME/bin:/usr/local/bin:$PATH

# Path to your oh-my-zsh installation.
export ZSH="/root/.oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="robbyrussell"

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )

# Uncomment the following line to use case-sensitive completion.
# CASE_SENSITIVE="true"

# Uncomment the following line to use hyphen-insensitive completion.
# Case-sensitive completion must be off. _ and - will be interchangeable.
# HYPHEN_INSENSITIVE="true"

# Uncomment the following line to disable bi-weekly auto-update checks.
# DISABLE_AUTO_UPDATE="true"

# Uncomment the following line to automatically update without prompting.
# DISABLE_UPDATE_PROMPT="true"

# Uncomment the following line to change how often to auto-update (in days).
# export UPDATE_ZSH_DAYS=13

# Uncomment the following line if pasting URLs and other text is messed up.
# DISABLE_MAGIC_FUNCTIONS="true"

# Uncomment the following line to disable colors in ls.
# DISABLE_LS_COLORS="true"

# Uncomment the following line to disable auto-setting terminal title.
# DISABLE_AUTO_TITLE="true"

# Uncomment the following line to enable command auto-correction.
# ENABLE_CORRECTION="true"

# Uncomment the following line to display red dots whilst waiting for completion.
# COMPLETION_WAITING_DOTS="true"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
# HIST_STAMPS="mm/dd/yyyy"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=( git 
zsh-syntax-highlighting
zsh-completions
zsh-autosuggestions
z
)
source $ZSH/oh-my-zsh.sh

# User configuration

# export MANPATH="/usr/local/man:$MANPATH"

# You may need to manually set your language environment
# export LANG=en_US.UTF-8

# Preferred editor for local and remote sessions
# if [[ -n $SSH_CONNECTION ]]; then
#   export EDITOR='vim'
# else
#   export EDITOR='mvim'
# fi

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# Set personal aliases, overriding those provided by oh-my-zsh libs,
# plugins, and themes. Aliases can be placed here, though oh-my-zsh
# users are encouraged to define aliases within the ZSH_CUSTOM folder.
# For a full list of active aliases, run `alias`.
#
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"
alias py="python"
alias py3="python3"
alias sqlmap="python /home/subv/sqlmap-dev/sqlmap.py"
alias docker="sudo docker"
alias dc="docker-compose"

alias setproxy="export ALL_PROXY=socks5://127.0.0.1:10808"
alias unsetproxy="unset ALL_PROXY"
```

### install.sh

vim打开 `:set ff=unix`

```bash
#!/bin/bash
#
cd ~/
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo apt update -y
sudo apt install zsh python2.7 python3-pip neovim nmap hydra  -y
sudo apt autoremove vim nano -y

pip3 install -i https://mirrors.ustc.edu.cn/pypi/web/simple pip -U
pip3 config set global.index-url https://mirrors.ustc.edu.cn/pypi/web/simple
sudo apt install python-dev -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt update -y
sudo apt install -y docker-ce
pip3 install docker-compose

sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \
       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

sudo mkdir ~/.config
sudo mkdir ~/.config/nvim/
sudo mkdir /etc/docker/
sudo mv ~/config/daemon.json /etc/docker/
sudo mv ~/config/init.vim ~/.config/nvim/
sudo apt install nodejs -y
sudo apt-get install yarn -y

sudo apt upgrade -y
vim +PlugInstall +qall

sudo rm -rf ~/.oh-my-zsh/
sudo rm -rf ~/.zshrc
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
echo "Do you want to set the git proxy(y/N)?"
read input
if [[ $input = "y" ]] || [[ $input = "Y" ]]; then
	host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
	git config --global http.https://github.com.proxy socks5://$host_ip:10808
	git config --global https.https://github.com.proxy socks5://$host_ip:10808
else
	echo "you can use this command by youself: git config --global https.https://github.com.proxy https://host_ip:port"
fi

sudo git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
sudo git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
sudo git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
cat ~/config/zshrc >> .zshrc

sudo git clone --depth=1 https://github.com/maurosoria/dirsearch.git /opt/dirsearch
sudo ln -sf /opt/dirsearch/dirsearch.py  /usr/local/bin/dirsearch

sudo git clone --depth=1 https://github.com/offensive-security/exploitdb.git /opt/exploitdb
sudo ln -sf /opt/exploitdb/searchsploit /usr/local/bin/searchsploit

sudo git clone --depth=1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

sudo git clone --depth=1 https://github.com/vulhub/vulhub.git

echo "input  :CocInstall coc-cmake  coc-git coc-highlight  coc-json  coc-python coc-sh  coc-snippets coc-syntax   in the vim command"

echo "========================================="
echo "=============Good, Enjoy it.============="
echo "========================================="
```

### WindowsTeminal

```json
// This file was initially generated by Windows Terminal Preview 1.5.3242.0
// It should still be usable in newer versions, but newer versions might have additional
// settings, help text, or changes that you will not see unless you clear this file
// and let us generate a new one for you.

// To view the default settings, hold "alt" while clicking on the "Settings" button.
// For documentation on these settings, see: https://aka.ms/terminal-documentation
{
    "$schema": "https://aka.ms/terminal-profiles-schema",

    "defaultProfile": "{2c4de342-38b7-51cf-b940-2309a097f518}",
   
    "initialCols": 81,           //终端窗口初始宽度
	"initialRows": 26,            //终端窗口初始高度
    // You can add more global application settings here.
    // To learn more about global settings, visit https://aka.ms/terminal-global-settings

    // If enabled, selections are automatically copied to your clipboard.
    "copyOnSelect": false,

    // If enabled, formatted data is also copied to your clipboard
    "copyFormatting": false,

    // A profile specifies a command to execute paired with information about how it should look and feel.
    // Each one of them will appear in the 'New Tab' dropdown,
    //   and can be invoked from the commandline with `wt.exe -p xxx`
    // To learn more about profiles, visit https://aka.ms/terminal-profile-settings
    "profiles":
    {
        "defaults":
        {
            // Put settings here that you want to apply to all profiles.
        },
        "list":
        [

            {
                "guid": "{2c4de342-38b7-51cf-b940-2309a097f518}",
                "hidden": false,
                "name": "Ubuntu",
                "source": "Windows.Terminal.Wsl",
                "fontFace" : "monaco",
               "padding" : "5, 5, 5, 5",
               "suppressApplicationTitle": true,
               "tabTitle": "Ubuntu",
               "acrylicOpacity": 0.7,
               "useAcrylic": true,
               "acrylicOpacity" : 0.70,
               "cursorColor": "#4ab118", //光标颜色
               "fontSize": 11,
               "snapOnInput" : true,
               "cursorShape": "filledBox", //光标形状
				"commandline": "ubuntu",
               "colorScheme" : "Ubuntu"
            },
            {
                // Make changes here to the powershell.exe profile.
                "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
                "name": "Windows PowerShell",
                "commandline": "powershell.exe",
                "hidden": false,
                "fontFace" : "monaco",
               "hidden": false,
               "acrylicOpacity": 0.7,
               "useAcrylic": true,
               "colorScheme" : "AdventureTime"
            },
            {
                // Make changes here to the cmd.exe profile.
                "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
                "name": "命令提示符",
                "commandline": "cmd.exe",
                "hidden": false,
                "fontFace" : "monaco",
               "padding" : "5, 5, 5, 5",
               "suppressApplicationTitle": false,
               "tabTitle": "CMD",
              "colorScheme" : "Ubuntu",
               "cursorColor": "#4ab118", //光标颜色
               "fontSize": 11,
               "snapOnInput" : true,
               "cursorShape": "filledBox",
               "startingDirectory": "D:\\Xray",
            },

            {
                "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
                "hidden": false,
                "name": "Azure Cloud Shell",
                "source": "Windows.Terminal.Azure"
            }
        ]
    },

    // Add custom color schemes to this array.
    // To learn more about color schemes, visit https://aka.ms/terminal-color-schemes
    "schemes": [              {
           "name": "Ubuntu",
           "black": "#2e3436",
           "red": "#cc0000",
           "green": "#2fff24",
           "yellow": "#c4a000",
           "blue": "#3465a4",
           "purple": "#75507b",
           "cyan": "#06989a",
           "white": "#d3d7cf",
           "brightBlack": "#555753",
           "brightRed": "#ef2929",
           "brightGreen": "#8ae234",
           "brightYellow": "#fce94f",
           "brightBlue": "#729fcf",
           "brightPurple": "#ad7fa8",
           "brightCyan": "#34e2e2",
           "brightWhite": "#eeeeec",
           "background": "#300a24",
           "foreground": "#eeeeec"
       },
       {
           "name": "AdventureTime",
           "black": "#050404",
           "red": "#bd0013",
           "green": "#4ab118",
           "yellow": "#e7741e",
           "blue": "#0f4ac6",
           "purple": "#665993",
           "cyan": "#70a598",
           "white": "#f8dcc0",
           "brightBlack": "#4e7cbf",
           "brightRed": "#fc5f5a",
           "brightGreen": "#9eff6e",
           "brightYellow": "#efc11a",
           "brightBlue": "#1997c6",
           "brightPurple": "#9b5953",
           "brightCyan": "#c8faf4",
           "brightWhite": "#f6f5fb",
           "background": "#1f1d45",
           "foreground": "#f8dcc0"
         }],

    // Add custom actions and keybindings to this array.
    // To unbind a key combination from your defaults.json, set the command to "unbound".
    // To learn more about actions and keybindings, visit https://aka.ms/terminal-keybindings
    "actions":
    [
        // Copy and paste are bound to Ctrl+Shift+C and Ctrl+Shift+V in your defaults.json.
        // These two lines additionally bind them to Ctrl+C and Ctrl+V.
        // To learn more about selection, visit https://aka.ms/terminal-selection
      { "command": {"action": "copy", "singleLine": false }, "keys": "ctrl+c" },
       { "command": "paste", "keys": "ctrl+v" },
       // Press Ctrl+Shift+F to open the search box
       { "command": "find", "keys": "ctrl+f" },
       { "command": { "action": "moveFocus", "direction": "down" }, "keys": "ctrl+down" },
       { "command": { "action": "moveFocus", "direction": "left" }, "keys": "ctrl+left" },
       { "command": { "action": "moveFocus", "direction": "right" }, "keys": "ctrl+right" },
       { "command": { "action": "moveFocus", "direction": "up" }, "keys": "ctrl+up" },
       { "command": { "action": "resizePane", "direction": "down" }, "keys": "alt+down" },
       { "command": { "action": "resizePane", "direction": "left" }, "keys": "alt+left" },
       { "command": { "action": "resizePane", "direction": "right" }, "keys": "alt+right" },
       { "command": { "action": "resizePane", "direction": "up" }, "keys": "alt+up" },
       { "command": "closePane", "keys": "ctrl+q" },

       { "command": { "action": "newTab", "index": 0 }, "keys": "ctrl+1" },
       { "command": { "action": "newTab", "index": 1 }, "keys": "ctrl+2" },
       { "command": { "action": "newTab", "index": 2 }, "keys": "ctrl+3" },

       // Press Alt+Shift+D to open a new pane.
       // - "split": "auto" makes this pane open in the direction that provides the most surface area.
       // - "splitMode": "duplicate" makes the new pane use the focused pane's profile.
       // To learn more about panes, visit https://aka.ms/terminal-panes
       { "command": { "action": "splitPane", "split": "auto", "splitMode": "duplicate" }, "keys": "ctrl+w" },
    ]
}
```

### VS Code

```json
{
    "files.autoSave": "onFocusChange",
    "editor.cursorStyle": "block",
    "editor.autoClosingBrackets": "never",
    "editor.fontSize": 16,
    "editor.suggestSelection": "first",
    "editor.rulers": [100],
    "workbench.colorCustomizations": {
                // 行号数字的颜色
        "editorLineNumber.foreground": "#17a346",
                // 光标的颜色
        "editorCursor.foreground": "#138136",

        "editor.selectionBackground": "#7c857f",
        "editor.background": "#413e3e",
        "activityBar.background": "#524f4f",
        "statusBar.background": "#524f4f",

        "activityBarBadge.background": "#2979FF",
        "activityBar.activeBorder": "#2979FF",
        "list.activeSelectionForeground": "#2979FF",
        "list.inactiveSelectionForeground": "#2979FF",
        "list.highlightForeground": "#2979FF",
        "scrollbarSlider.activeBackground": "#2979FF50",
        "editorSuggestWidget.highlightForeground": "#2979FF",
        "textLink.foreground": "#2979FF",
        "progressBar.background": "#2979FF",
        "pickerGroup.foreground": "#2979FF",
        "tab.activeBorder": "#2979FF",
        "notificationLink.foreground": "#2979FF",
        "editorWidget.resizeBorder": "#2979FF",
        "editorWidget.border": "#2979FF",
        "settings.modifiedItemIndicator": "#2979FF",
        "settings.headerForeground": "#2979FF",
        "panelTitle.activeBorder": "#2979FF",
        "breadcrumb.activeSelectionForeground": "#2979FF",
        "menu.selectionForeground": "#2979FF",
        "menubar.selectionForeground": "#2979FF",
        "editor.findMatchBorder": "#2979FF",
        "selection.background": "#13cc7040",
        "statusBarItem.remoteBackground": "#2979FF"
    },
    "materialTheme.accent": "Blue",
    "terminal.integrated.fontFamily": "monaco",
    "editor.fontFamily": "Fira Code",
    "editor.fontLigatures": true,
    "editor.cursorBlinking": "phase",
    "editor.fontWeight": "490",
    "editor.autoClosingQuotes": "never",
    "editor.autoSurround": "never",
    "editor.renderIndentGuides": false,
    "workbench.tree.renderIndentGuides": "none",
    "editor.highlightActiveIndentGuide": false,
    "editor.copyWithSyntaxHighlighting": false,
    "editor.renderLineHighlight": "none",
    "workbench.editorAssociations": {
        "*.ipynb": "jupyter-notebook"
    },
    "workbench.colorTheme": "Seti Monokai",
    "workbench.editor.untitled.hint": "hidden",
    "workbench.iconTheme": "vscode-icons",
    "workbench.startupEditor": "none",
    "notebook.cellToolbarLocation": {
        "default": "right",
        "jupyter-notebook": "left"
    },

    "editor.tokenColorCustomizations": {
        "textMateRules": [
          {
            "name": "Comment",
            "scope": [
              "comment",
              "comment.block",
              "comment.block.documentation",
              "comment.line",
              "comment.line.double-slash",
              "punctuation.definition.comment",
            ],
            "settings": {
              "fontStyle": "",
              //斜体 "fontStyle": "italic",
              //斜体下划线 "fontStyle": "italic underline",
              //斜体粗体下划线 "fontStyle": "italic bold underline",
            }
          },
        ]
    },
    "workbench.settings.editor": "json",
}
```

### Sublimetext3

```json
{
	"auto_complete": true,
	"auto_match_enabled": false,
	"color_scheme": "Packages/Color Scheme - Default/Monokai.sublime-color-scheme",
	"dictionary": "Packages/Language - English/en_GB.dic",
	"font_face": "monaco",
	"font_size": 12,
	"ignored_packages":
	[
 "Vintage"
	]
}

bracket:
{
   "bracket_styles": {
       // `default` and `unmatched` styles are special
       // styles. If they are not defined here,
       // they will be generated internally with
       // internal defaults.
       // `default` style defines attributes that
       // will be used for any style that does not
       // explicitly define that attribute.  So if
       // a style does not define a color, it will
       // use the color from the "default" style.
       "default": {
           "icon": "dot",
           // Support the old convention of `brackethighlighter.default`
           // for themes that already provide something.
           // As this has always been the only one we've provided
           // by default, all the others will use region-ish colors.
           "color": "region.yellowish brackethighlighter.default",
           // "style": "underline"
           "style": "hightlight"
       },
       // This particular style is used to highlight
       // unmatched bracket pairs.  It is a special
       // style.
       "unmatched": {
           "icon": "question",
           "color": "region.redish",
           "style": "outline"
       },
       // User defined region styles
       "curly": {
           "icon": "curly_bracket",
           "color": "region.purplish",
           "style": "hightlight"
       },
       "round": {
           "icon": "round_bracket",
           "color": "region.yellowish",
           "style": "hightlight"
       },
       "square": {
           "icon": "square_bracket",
           "color": "region.bluish",
           "style": "hightlight"
       },
       //下面的都是原来的样式
       "angle": {
           "icon": "angle_bracket",
           "color": "region.orangish"
           // "style": "underline"
       },
       "tag": {
           "icon": "tag",
           "color": "region.orangish"
           // "style": "underline"
       },
       "c_define": {
           "icon": "hash",
           "color": "region.yellowish"
           // "style": "underline"
       },
       "single_quote": {
           "icon": "single_quote",
           "color": "region.greenish"
           // "style": "underline"
       },
       "double_quote": {
           "icon": "double_quote",
           "color": "region.greenish"
           // "style": "underline"
       },
       "regex": {
           "icon": "star",
           "color": "region.greenish"
           // "style": "underline"
       }
   }
}
```

[Adb卸载手机原装软件](https://www.notion.so/Adb-b446584bb3f34dbcab6f007c52edb681)

