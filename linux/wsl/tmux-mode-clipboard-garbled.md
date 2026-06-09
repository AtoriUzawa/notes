# WSL + tmux 复制中文 / Nerd Font 字符到 Windows 乱码排查

## 问题现象

WSL + tmux 中复制中文或 Nerd Font 字符，粘贴到 Windows 出现乱码。

---

## 排查过程

### 1. Locale 正常

```bash
locale
```

```text
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8
```

结论：UTF-8 环境正常。

---

### 2. tmux 配置正常

```tmux
set -g default-terminal "tmux-256color"
set -ga terminal-overrides ",*:Tc"
```

结论：tmux 终端能力配置无异常。

---

### 3. 字符显示正常（排除编码/字体问题）

```bash
printf "中文 ✓ ❯ \n"
```

* tmux 内正常
* tmux 外正常

结论：

* 非 UTF-8 问题
* 非字体问题
* 非 tmux 渲染问题

---

### 4. Windows 剪贴板正常

```bash
printf "❯ 中文测试" | iconv -f utf8 -t utf16le | clip.exe
```

Windows 粘贴正常。

结论：

* clip.exe 正常
* Windows 剪贴板正常

---

## 初步结论

问题不在编码 / 字体 / tmux 显示，而更可能在：

* tmux copy-pipe 处理链
* 或 copy-mode 绑定方式

---

## 可尝试修复方案（使用此方案请移除 tmux-yank 插件）

### 使用 UTF-16LE + clip.exe

```tmux
bind -T copy-mode-vi y \
send-keys -X copy-pipe-and-cancel \
"iconv -f utf8 -t utf16le | clip.exe"
```

---

### 重新加载配置

```bash
tmux source-file ~/.config/tmux/tmux.conf
```
