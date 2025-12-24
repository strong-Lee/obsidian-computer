```json
{
  // 整个设计系统的根对象
  "designSystem": {
    // ----------------------------------------------------------------------
    // 1. 元数据 (Metadata): 关于设计系统的基本信息，用于文档和版本管理
    // ----------------------------------------------------------------------
    "metadata": {
      "name": "Serene Wellness App Design System", // 设计系统名称
      "version": "1.0", // 版本号
      "author": "AI Super-Collective Persona", // 作者
      "description": "A design system focused on creating a warm, premium, natural, and accessible user experience for a wellness application." // 描述
    },
    // ----------------------------------------------------------------------
    // 2. 设计哲学 (Philosophy): 提炼出的核心设计原则
    // ----------------------------------------------------------------------
    "philosophy": {
      "brandTone": "Warm, Premium, Textured, Natural", // 品牌调性：温暖、高级、有质感、自然
      "principles": [ // 核心原则列表
        "Serenity First: Prioritize breathability and avoid visual noise.", // 宁静优先：优先考虑呼吸感，避免视觉噪音
        "Organic Interaction: Motion should be meaningful, physical, and guide the user.", // 有机交互：动效应有意义、模拟物理世界并引导用户
        "Tactile Texture: Evoke physical materials like paper and clay through visual design.", // 触感纹理：通过视觉设计唤起纸张、粘土等物理材质的感觉
        "Clarity & Accessibility: Beauty must not compromise usability." // 清晰与无障碍：美观不能牺牲可用性
      ]
    },
    // ----------------------------------------------------------------------
    // 3. 设计变量/令牌 (Design Tokens): 整个系统的核心！
    // ----------------------------------------------------------------------
    "tokens": {
      // 3.1 颜色 (Colors)
      "colors": {
        // --- 原始调色板 (Primitive Palette) ---
        // 这里定义了“原子颜色”，它们是色值本身，不关心用在哪里。这是“What”。
        "almondMilk": "#FDF9F6",     // 杏仁牛奶 (你的主背景)
        "oatmeal": "#F1EBE4",        // 燕麦色 (你的次背景/卡片)
        "oatmealHover": "#E9E2DC",   // 燕麦色-悬停
        "warmGray": "#F5F5F5",       // 暖灰色 (你的禁用背景)
        "deepBrownGray": "#4E4A47",  // 深棕灰 (你的主文字)
        "lightBrownGray": "#6D6661", // 浅棕灰 (你的次文字)
        "paleBrownGray": "#A8A29E",  // 灰褐色 (你的三级/占位符文字)
        "neutralGray": "#BDBDBD",    // 中性灰 (你的禁用文字)
        "terracotta": "#D8AFA0",      // 赤陶色 (你的主操作色/强调色)
        "terracottaDark": "#C99E8A",  // 赤陶色-深 (主操作色悬停)
        "sageGreenLight": "#E6F5EC", // 鼠尾草绿-浅 (成功状态背景)
        "sageGreenDark": "#5A8E70",  // 鼠尾草绿-深 (成功状态文字)
        "roseDustLight": "#FBE9E9",  // 玫瑰豆沙-浅 (错误状态背景)
        "roseDustDark": "#C77070",   // 玫瑰豆沙-深 (错误状态文字)
        "infoBlueLight": "#E3EAF3",  // 信息蓝-浅 (信息状态背景)
        "infoBlueDark": "#6B82A5",   // 信息蓝-深 (信息状态文字)
        "mistGray": "#DCD9D5",       // 雾灰色 (你的边框颜色)

        // --- 语义化Token (Semantic Tokens) ---
        // 这里定义了颜色的“用途”，它引用上面的原子颜色。这是“How”和“Where”。
        // 这种结构让主题切换变得非常容易。
        "background": { // 背景色
          "primary": "{colors.almondMilk}",    // 主背景 (画布)
          "secondary": "{colors.oatmeal}",     // 次背景 (卡片、容器)
          "hover": "{colors.oatmealHover}",    // 悬停背景 (列表项等)
          "disabled": "{colors.warmGray}"      // 禁用背景
        },
        "text": { // 文字颜色
          "primary": "{colors.deepBrownGray}",   // 主要文字
          "secondary": "{colors.lightBrownGray}",// 次要文字
          "tertiary": "{colors.paleBrownGray}",  // 三级文字 (用于占位符 placeholder)
          "disabled": "{colors.neutralGray}",    // 禁用文字
          "onAccent": "{colors.almondMilk}"      // 在强调色背景上的文字 (反色)
        },
        "action": { // 操作相关颜色
          "primary": { // 主要操作 (如主要按钮)
            "background": "{colors.terracotta}", // 背景色
            "hover": "{colors.terracottaDark}",  // 悬停背景色
            "text": "{colors.almondMilk}"        // 文字颜色 (🚨注意：这里是浅色，与你中文稿的深色不同)
          },
          "secondary": { // 次要操作 (如次要按钮)
            "border": "{colors.terracotta}",     // 边框颜色
            "text": "{colors.terracottaDark}",   // 文字颜色
            "hoverBackground": "{colors.oatmealHover}" // 悬停时的背景填充色
          }
        },
        "border": { // 边框颜色
          "primary": "{colors.mistGray}",   // 主要边框/分割线
          "focused": "{colors.terracotta}"  // 激活状态的边框 (如输入框)
        },
        "feedback": { // 反馈色 (成功、错误、信息)
          "success": { "background": "{colors.sageGreenLight}", "text": "{colors.sageGreenDark}" },
          "error": { "background": "{colors.roseDustLight}", "text": "{colors.roseDustDark}" },
          "info": { "background": "{colors.infoBlueLight}", "text": "{colors.infoBlueDark}" }
        }
      },
      // 3.2 字体 (Typography)
      "typography": {
        "fontFamily": { // 字体族
          "serif": "'FangZheng SongKeBen XiuKai', serif", // 衬线字体 (具体化了你的Serif)
          "sans": "'PingFang SC', 'Source Han Sans', 'Noto Sans CJK SC', 'HarmonyOS Sans', sans-serif" // 非衬线字体
        },
        "scale": { // 字号、字重、行高体系
          "h1": { "fontSize": "64rpx", "fontWeight": "500", "lineHeight": "1.4", "fontFamily": "{typography.fontFamily.serif}" }, // 页面大标题
          "h2": { "fontSize": "48rpx", "fontWeight": "500", "lineHeight": "1.4", "fontFamily": "{typography.fontFamily.serif}" }, // 模块标题
          "h3": { "fontSize": "36rpx", "fontWeight": "500", "lineHeight": "1.5", "fontFamily": "{typography.fontFamily.serif}" }, // 小节标题
          "bodyLarge": { "fontSize": "32rpx", "fontWeight": "400", "lineHeight": "1.7", "fontFamily": "{typography.fontFamily.sans}" }, // 大正文 (JSON新增)
          "bodyRegular": { "fontSize": "28rpx", "fontWeight": "400", "lineHeight": "1.7", "fontFamily": "{typography.fontFamily.sans}" }, // 常规正文
          "caption": { "fontSize": "24rpx", "fontWeight": "400", "lineHeight": "1.6", "fontFamily": "{typography.fontFamily.sans}" }, // 辅助/说明文字
          "button": { "fontSize": "30rpx", "fontWeight": "500", "lineHeight": "1.4", "fontFamily": "{typography.fontFamily.sans}" } // 按钮文字
        }
      },
      // 3.3 间距 (Spacing) - 基于8rpx网格系统
      "spacing": {
        "base": "8rpx", // 基础单位
        "xxs": "4rpx", "xs": "8rpx", "s": "16rpx", "m": "24rpx", "l": "32rpx", "xl": "40rpx", "xxl": "48rpx"
      },
      // 3.4 圆角 (Radius)
      "radius": {
        "s": "8rpx", "m": "16rpx", "l": "24rpx", "xl": "32rpx", // 不同尺寸的圆角
        "full": "50%", // 全圆 (用于头像)
        "pill": "999px" // 胶囊形状
      },
      // 3.5 阴影 (Shadows)
      "shadows": {
        "subtle": "0 2rpx 6rpx 0 rgba(78, 74, 71, 0.05)", // 一个更微妙的阴影
        "card": "0 2rpx 6rpx 0 rgba(78, 74, 71, 0.05), 0 8rpx 24rpx 0 rgba(78, 74, 71, 0.08)", // 卡片阴影 (多层)
        "button": "0 4rpx 12rpx 0 rgba(216, 175, 160, 0.3)" // 按钮阴影
      },
      // 3.6 动效 (Motion)
      "motion": {
        "easing": "cubic-bezier(0.4, 0, 0.2, 1)", // 缓动曲线
        "duration": { // 时长
          "short": "200ms", // 短时长 (用于微交互)
          "medium": "350ms", // 中等时长 (用于组件过渡)
          "long": "450ms"  // 长时长 (用于页面过渡)
        }
      }
    },
    // ----------------------------------------------------------------------
    // 4. 组件 (Components): 将上面的Tokens组合起来，定义具体组件的样式
    // ----------------------------------------------------------------------
    "components": {
      // 4.1 按钮 (Button)
      "button": {
        "base": { // 所有按钮的通用基础样式
          "height": "88rpx",
          "borderRadius": "{radius.l}", // 引用圆角Token
          "typography": "{typography.scale.button}", // 引用字体Token
          "transition": "background-color {motion.duration.short} {motion.easing}, transform {motion.duration.short} {motion.easing}" // 引用动效Token
        },
        "variants": { // 不同类型的按钮
          "primary": { // 主要按钮
            "background": "{colors.action.primary.background}",
            "color": "{colors.action.primary.text}",
            "boxShadow": "{shadows.button}",
            "states": { // 各种状态
              "hover": { "background": "{colors.action.primary.hover}" }, // 悬停状态
              "pressed": { "background": "{colors.action.primary.hover}", "transform": "scale(0.98)" }, // 按下状态
              "disabled": { "background": "{colors.background.disabled}", "color": "{colors.text.disabled}", "boxShadow": "none" } // 禁用状态
            }
          },
          "secondary": { // 次要按钮
            "background": "transparent",
            "color": "{colors.action.secondary.text}",
            "border": "2rpx solid {colors.action.secondary.border}",
             "states": {
              "hover": { "background": "{colors.action.secondary.hoverBackground}" },
              "pressed": { "background": "{colors.action.secondary.hoverBackground}", "transform": "scale(0.98)" },
              "disabled": { "background": "transparent", "color": "{colors.text.disabled}", "borderColor": "{colors.border.primary}" }
            }
          }
        }
      },
      // 4.2 输入框 (Input)
      "input": {
        "base": {
          "height": "96rpx",
          "background": "{colors.background.secondary}",
          "color": "{colors.text.primary}",
          "border": "2rpx solid {colors.border.primary}",
          "borderRadius": "{radius.m}",
          "typography": "{typography.scale.bodyRegular}",
          "padding": "0 {spacing.s}"
        },
        "states": { // 输入框的各种状态
          "hover": { "borderColor": "{colors.text.secondary}" }, // 悬停
          "focus": { "borderColor": "{colors.border.focused}", "boxShadow": "0 0 0 3rpx rgba(216, 175, 160, 0.2)" }, // 激活 (增加了辉光效果)
          "error": { "borderColor": "{colors.feedback.error.text}" }, // 错误
          "disabled": { "background": "{colors.background.disabled}", "color": "{colors.text.disabled}", "borderColor": "transparent" } // 禁用
        }
      },
      // 4.3 卡片 (Card)
      "card": {
        "base": {
          "background": "{colors.background.secondary}",
          "borderRadius": "{radius.l}",
          "padding": "{spacing.l}", // 内边距
          "boxShadow": "{shadows.card}"
        }
      },
      // 4.4 模态框 (Modal)
      "modal": {
        "overlay": { "background": "rgba({colors.deepBrownGray}, 0.4)" }, // 遮罩层 (这里引用颜色有问题，应该是"rgba(78, 74, 71, 0.4)")
        "container": { // 弹窗容器
          "background": "{colors.background.primary}",
          "borderRadius": "{radius.xl}",
          "boxShadow": "{shadows.card}"
        }
      },
      // 4.5 加载指示器 (Loader)
      "loader": {
        "spinner": { // 旋转加载
          "description": "一个由三个点组成的呼吸动画。",
          "dots": {
            "color": "{colors.action.primary.background}",
            "count": 3,
            "animation": "breathing ease-in-out infinite"
          }
        },
        "skeleton": { // 骨架屏
          "shapeBackground": "{colors.background.secondary}", // 骨架形状的背景
          "shimmerEffect": { // 闪光效果
            "background": "linear-gradient(90deg, transparent, {colors.background.hover}, transparent)" // 从左到右的扫光动画
          }
        }
      }
    }
  }
}
```