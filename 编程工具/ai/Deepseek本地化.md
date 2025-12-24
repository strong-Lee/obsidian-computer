**一、环境准备**

```

# 1. 安装Xcode命令行工具（必须）

xcode-select --install

  

# 2. 安装Homebrew包管理器

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc

source ~/.zshrc

brew update # 更新最新版本

  

# 3. 安装必需组件

brew install cmake git python@3.10

brew install git-lfs # git lfs --version

  

  

# 4. 创建专用虚拟环境（避免污染系统环境）

mkdir ~/deepseek_deploy && cd ~/deepseek_deploy

python3.10 -m venv .venv

source .venv/bin/activate

pip install torch

// 一个开源的深度学习框架，用于构建和训练神经网络。它由 Facebook 的人工智能研究团队开发，

// 支持 **动态图**（动态计算图）和 **自动微分**，这使得它非常适合研究和实验。

pip install transformers

//由 **Hugging Face** 提供的一个库，主要用于自然语言处理（NLP）任务。

// 它提供了对预训练语言模型（如 GPT、BERT、T5、BART、Llama 等）的高效访问和集成，

// 支持简单的接口用于训练和推理。

pip install huggingface-hub

// 这是一个用于与 **Hugging Face** 模型库交互的工具，允许用户方便地上传、下载和管理模型。

// 它为用户提供了一个中心化的平台，用于保存和分享机器学习模型。

```

  

**二、编译LLama.cpp（针对Inter Mac优化）**

**方法一**

```

# 1. 克隆仓库

git clone https://github.com/ggerganov/llama.cpp

cd llama.cpp

  

# 2. 使用Metal兼容模式编译（即使没有GPU也需开启）

make LLAMA_METAL=1 CC=/usr/bin/clang CXX=/usr/bin/clang++

  

# 3. 验证编译结果（应显示帮助信息）

./main -h

```

**方法二**

```

brew search llama

brew install llama.cpp

```

 三、获取Deepseek模型

从huggingface下载模型

```

git lfs install

git clone https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B

```