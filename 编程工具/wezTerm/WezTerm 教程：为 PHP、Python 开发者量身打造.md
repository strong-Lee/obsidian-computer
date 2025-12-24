WezTerm 是一个由 GPU 加速的、跨平台的终端模拟器和多路复用器（Multiplexer）。你可以把它想象成 **iTerm2/Terminal.app（终端模拟器）** 和 **tmux/screen（多路复用器）** 的结合体，但全部功能都内置并且用 Lua 语言进行配置。

**对你而言最重要的特点：**

1. **GPU 加速渲染**: 在你的 Radeon Pro 560 显卡上，滚动、输出大量日志、运行复杂的 TUI (终端界面) 应用（如 lazygit）会如丝般顺滑，告别卡顿。
    
2. **内置多路复用 (Multiplexing)**: 你可以在一个 WezTerm 窗口中创建多个**标签页 (Tabs)**，每个标签页中可以分割出多个**窗格 (Panes)**。这对于同时运行后端服务、前端构建、数据库和测试等任务至关重要，无需再额外学习和配置 tmux。
    
3. **强大的 Lua 配置**: 几乎所有功能——从外观、快捷键到复杂的行为逻辑——都通过一个 wezterm.lua 文件进行配置。这提供了无与伦比的灵活性。
    
4. **智能功能**:
    
    - **连字 (Ligatures)**: 自动将 ->、=>、!= 等编程符号合并成更美观的单个符号。
        
    - **点击链接**: 自动识别 URL、文件路径，并允许你按住 Cmd 点击打开。
        
    - **语义分区 (Semantic Zones)**: 能够识别命令的开始和结束，方便你快速选中整个命令的输出。
        
5. **内置 SSH 客户端**: 可以像管理本地 Shell 一样管理远程 SSH 会话，简化远程开发流程。
    
6. **跨平台**: 如果你偶尔需要在 Windows 或 Linux 上工作，可以拥有一套完全一致的终端体验。

#### 安装 WezTerm

```bash
brew install --cask wezterm
```

```markdown-tree
    second
        third
            fourth
                file1.jpg
                file2.txt
                file3.pdf
            abc

```

```bash
# 安装 Nerd Font 字体 (对图标显示至关重要) 
brew tap homebrew/cask-fonts 
brew install --cask font-hack-nerd-font 

brew install lsd # 安装 lsd (一个带有图标的 ls 命令替代品) 
brew install zoxide  # 安装 zoxide (一个更智能的 cd 命令) 
brew install btop  # 安装 btop (资源监视器) 
brew install neofetch # 安装 neofetch (用于显示系统信息的工具)
```

```bash
git clone --recursive https://github.com/strong-Lee/my_wezterm.git ~/.config/wezterm
```