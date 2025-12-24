```bash
# ~/.config/fish/config.fish

# ===================================================================
# 1. 环境变量 (Environment Variables)
# ===================================================================
# 使用 fish_add_path 来管理 PATH，这是 Fish 的推荐方式。
# 它会自动避免重复添加，比手动 'set -gx PATH ...' 更安全、更清晰。

# 手动添加 Homebrew 路径 (适用于 Intel Mac)
fish_add_path /usr/local/bin
fish_add_path /usr/local/sbin

# Composer 全局包路径 (用于 PHP LS 等工具)
fish_add_path ~/.composer/vendor/bin

# PHP 特定版本路径
fish_add_path /usr/local/opt/php@7.4/bin
fish_add_path /usr/local/opt/php@7.4/sbin

# 用户自定义脚本路径
fish_add_path ~/bin

# 设置代理 (使用 -U 或 --universal 来使其在所有 fish 会话中持久化)
set -Ux http_proxy http://127.0.0.1:7897
set -Ux https_proxy http://127.0.0.1:7897

# ===================================================================
# 2. 别名 (Aliases)
# ===================================================================
# 将常用命令简化
alias vim="nvim"
alias ls="ls -G" # 在 macOS 上启用颜色
alias ll="ls -lGh"
alias la="ls -lGha"

# SSH 快捷方式
alias ailaTS="ssh aila@192.168.1.19"
alias BXT="ssh root@120.26.246.116"

# ===================================================================
# 3. 工具和插件初始化
# ===================================================================

# Starship Prompt: 一个漂亮的、可定制的命令提示符
eval (starship init fish)

# fzf: 命令行模糊搜索器
# --fish 选项会输出必要的 fish shell 配置代码
fzf --fish | source

# zoxide: 一个更智能的 `cd` 命令
zoxide init fish | source

# autojump: 另一个目录快速跳转工具 (如果你同时用 zoxide 和 autojump，可以考虑只留一个)
if test -e (brew --prefix autojump)/share/autojump/autojump.fish
    source (brew --prefix autojump)/share/autojump/autojump.fish
end

# bobthefish 主题配置 (如果你使用 starship, 这部分可以注释掉或删除，因为 starship 会接管提示符)
# set -g theme_nerd_fonts yes
# set -g theme_color_scheme nord

# ===================================================================
# 4. FZF 定制化
# ===================================================================
# FZF 默认选项
set -x FZF_DEFAULT_OPTS '--height 80% --layout=reverse --border --multi'
# 使用 fd 作为 fzf 的默认文件查找命令，速度更快
set -x FZF_DEFAULT_COMMAND 'fd --type f --hidden --follow --exclude .git'

# ===================================================================
# 5. 函数与按键绑定
# ===================================================================
# fish 会自动调用 `fish_user_key_bindings` 函数来设置快捷键
# 将所有 `bind` 命令放在这里是最佳实践
function fish_user_key_bindings
    # Ctrl+T: 使用 fzf 查找文件
    # 我们将预览功能直接绑定在这里
    bind \ct "fzf --preview 'bat --style=numbers --color=always --line-range :500 {}' | read -l cmd; and commandline -- -i \$cmd"

    # Ctrl+R: 使用 fzf 搜索历史命令
    bind \cr "history | fzf | read -l cmd; and commandline -- -r \$cmd"
end

# ===================================================================
# 6. SSH Agent
# ===================================================================
# 自动启动 ssh-agent 并在需要时添加密钥
if test -z "$SSH_AUTH_SOCK"
    eval (ssh-agent -c)
    # ssh-add # 如果你希望每次启动都自动添加所有默认密钥，可以取消此行注释
end
```
#lazyvim #fish