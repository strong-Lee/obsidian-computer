```lua
local wezterm = require('wezterm')
local platform = require('utils.platform')

-- local font = 'JetBrainsMono Nerd Font'
-- local font = 'Fira Mono for Powerline'
local font_size = platform().is_mac and 16 or 9

return {
   -- 将我们安装的 Nerd Font 作为第一选择，这样才能正确显示图标。
   -- 当遇到 Emoji 时，会自动回退到 Apple Color Emoji。
   -- font = wezterm.font(font),
   font = wezterm.font_with_fallback({
      'JetBrainsMono Nerd Font',
      'Apple Color Emoji',
      -- 'Fira Mono for Powerline',
   }),
   font_size = font_size,
   line_height = 1,

   --ref: https://wezfurlong.org/wezterm/config/lua/config/freetype_pcf_long_family_names.html#why-doesnt-wezterm-use-the-distro-freetype-or-match-its-configuration
   freetype_load_target = 'Normal', -- -@type 'Normal'|'Light'|'Mono'|'HorizontalLcd'
   freetype_render_target = 'Normal', -- -@type 'Normal'|'Light'|'Mono'|'HorizontalLcd'

   -- 禁用字体抗锯齿（可选）
   --font_antialias = 'None',
   --font_hinting = 'None',

   -- 调整刷新率（提高刷新率可能有助于减少闪烁）
   --front_end = 'OpenGL',
   --animation_fps = 60,
   harfbuzz_features = {
      'calt',
      'liga',
      'ss01',
      'ss02',
      'ss04',
      'ss05',
      'ss06',
      'ss07',
      'ss08',
      'ss10',
      'ss11',
      'ss12',
      'ss17',
      'ss18',
   },
}
```
#lua #wezterm