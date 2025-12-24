	环境就绪，现在我们来请主角登场。

### 步骤 1：安装 LazyVim Starter 模板

> [!WARNING]  
> 下面的命令会备份并替换你现有的 Neovim 配置。如果你是全新安装，可以放心执行。

```bash
# 备份旧配置，安全第一 (如果存在的话)
mv ~/.config/nvim ~/.config/nvim.bak
mv ~/.local/share/nvim ~/.local/share/nvim.bak
mv ~/.local/state/nvim ~/.local/state/nvim.bak
mv ~/.cache/nvim ~/.cache/nvim.bak

# 从 GitHub 克隆 LazyVim 的入门模板到标准配置路径
git clone https://github.com/LazyVim/starter ~/.config/nvim

# 移除模板的 Git 历史，让它从现在开始成为你自己的项目
rm -rf ~/.config/nvim/.git
```

### 步骤 2：首次启动与插件管理

	现在，在终端里输入 nvim 并回车。

	第一次启动时，你会看到一个漂亮的启动界面，lazy.nvim 正在自动下载并安装所有预设的插件。稍等片刻，安装完成后按 q 退出，再次进入 nvim。

- **核心理念**: **懒加载 (Lazy Loading)**。插件只在被触发时（例如打开特定文件类型、执行某个命令）才加载，这使得 Neovim 的启动和响应速度飞快。
    
- **目录结构**:
    
    - ~/.config/nvim/lua/config/: 存放核心配置。
        
        - lazy.lua: lazy.nvim 插件管理器的配置。
            
        - options.lua: Neovim 的全局选项 (:h options)。
            
        - keymaps.lua: 全局快捷键绑定。
            
    - ~/.config/nvim/lua/plugins/: **这是我们之后最常打交道的地方**。每个 .lua 文件代表一个或一组插件的配置。LazyVim 会自动加载这个目录下的所有 .lua 文件。

- **插件管理中心**: 在 Normal 模式下输入 :Lazy 并回车，打开管理界面。

	- U: 一键**U**pdate（更新）所有插件。
	- X: 清理不再使用的插件。
	- c: 查看选中插件的详细**c**onfig（配置）。
	- q: **Q**uit（退出）管理界面。

### 步骤 3：核心快捷键 (你的超能力)

	LazyVim 的“魔法棒”是 <Space> (空格键)，也叫 `<Leader`> 键。所有强大的功能都由它发起。

快捷键|功能|记忆技巧 (Mnemonic)
-- | -- | --
<Space> f f | Find Files (查找项目中的文件)|最常用，肌肉记忆！
<Space> f g | Find Grep (全局搜索文本)|搜索Grep
<Space> f w | Find Word (全局搜索光标下的单词)|搜索Word
<Space> b b|切换已打开的文件 (Buffers | Buffer Browser
<Space> f o|Find Oldfiles (查找最近打开的文件)|查找Old
<Space> ;|查找Commands (命令)|（新版 LazyVim 可能是 <Space> :）
gd|Go to Definition (跳转到定义)|
gr|Go to References (查找所有引用)|
<Space> s . 或 <leader> f c|查找工作区Symbols (类/函数/变量Symbols / Find Class
<Space> e|打开/关闭左侧的Explorer (文件树Explorer
<Space> /|打开/关闭浮动终端 | 像在 Vim 中搜索一样，但打开的是终端
<Space> g g|打开 Git 界面 Go to Git)

### 步骤 4：换个更酷的主题 (Catppuccin)

默认的 tokyonight 很棒，但让我们来试试另一个超流行的主题。

1. 进入你的插件配置目录：
```bash
cd ~/.config/nvim/lua/plugins
```

2. 创建一个新的配置文件来存放主题设置：
```bash
touch colorscheme.lua
```
3. 用 nvim [[lazyvim_colorscheme.lua]] 打开这个新文件，然后进行操作。

		保存退出 (:wq)。现在，重新启动 nvim，你会看到一个全新的、柔和而漂亮的 Catppuccin 主题！

---

下一章：[[003：PHP 生态深度集成 —— 打造专属工作利器]]

#lazyvim 