# iTerm2 AI Presenter - 设计文档

## 项目目标
在 iTerm2 的独立 tab 中实现一个实时 AI 讲解员（Coding Waifu），可以：
- 显示动画角色（waifu）
- 实时语音讲解 Claude Code 的编程进展
- 显示同步字幕

## 技术方案

### 显示效果设计

```
┌─────────────────────────────────────────┐
│  iTerm2 Tab 2: Coding Waifu Presenter   │
├─────────────────────────────────────────┤
│                                         │
│        [Waifu 动画 GIF/图片]            │
│                                         │
├─────────────────────────────────────────┤
│  💬 实时字幕区域                        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  "嗨~我注意到 Claude 刚开始分析         │
│   authentication.ts 文件呢！             │
│   看起来是要优化登录流程 ✨"             │
│                                         │
│  🎤 [语音播放中...]                     │
│                                         │
│  📊 当前进度：                           │
│  ├─ ✅ 读取文件                         │
│  ├─ 🔄 分析代码结构                     │
│  ├─ ⏳ 生成优化方案                     │
│  └─ ⏳ 应用修改                         │
└─────────────────────────────────────────┘
```

### 核心功能模块

#### 1. 监控模块 (Monitor)
- **方式 A**: iTerm2 Python API - 监听 Claude Code tab 的输出
- **方式 B**: 文件监听 - 监听 Claude Code 的日志文件
- **方式 C**: tmux 集成 - 通过 tmux capture-pane 读取内容

#### 2. 智能分析模块 (Analyzer)
```python
# 识别关键事件
events = {
    "task_start": "开始新任务",
    "file_read": "读取文件",
    "file_edit": "编辑文件",
    "bash_command": "执行命令",
    "error": "遇到错误",
    "task_complete": "完成任务",
    "git_commit": "提交代码"
}
```

#### 3. 讲解生成模块 (Narrator)
- **文本生成**：
  - 选项 A：预定义模板 + 参数替换（快速）
  - 选项 B：调用 LLM API 生成自然讲解（更生动）

- **示例模板**：
```python
templates = {
    "task_start": [
        "好的，开始新任务了~让我看看要做什么！",
        "哇，有新工作来了！Claude 要开始写代码啦~",
        "注意注意！新的编程任务开始了呢！"
    ],
    "file_edit": [
        "Claude 正在修改 {filename}，看起来很认真呢~",
        "哦哦！正在编辑 {filename}，写了 {lines} 行代码！",
        "让我看看...{filename} 被更新了，期待新功能！"
    ]
}
```

#### 4. TTS 语音模块 (Speech)
**选项对比：**

| 方案 | 优势 | 劣势 | 适用场景 |
|------|------|------|----------|
| macOS `say` | 免费、本地、快速 | 声音不够自然 | 原型开发 |
| OpenAI TTS | 声音自然、多语言 | 需要 API key、有成本 | 生产环境 |
| Azure TTS | 高质量、支持 SSML | 配置复杂 | 企业级 |
| Coqui TTS | 本地运行、免费 | 需要 GPU、延迟高 | 离线场景 |

**推荐**：开发阶段用 `say`，后期升级到 OpenAI TTS

#### 5. 数字人显示模块 (Avatar)
**实现方式：**

##### 方式 1：GIF 动画序列（推荐用于原型）
```
assets/waifu/
├── idle.gif          # 待机动画
├── thinking.gif      # 思考中
├── typing.gif        # 编码中
├── happy.gif         # 完成任务
├── confused.gif      # 遇到错误
└── celebrating.gif   # 庆祝
```

##### 方式 2：静态图片序列
```python
# 根据情绪切换图片
emotions = {
    "neutral": "waifu_neutral.png",
    "happy": "waifu_happy.png",
    "thinking": "waifu_thinking.png",
    "error": "waifu_sad.png"
}
```

##### 方式 3：Live2D（高级方案）
- 使用 Live2D Cubism
- 导出为视频帧或 GIF
- 或通过浏览器渲染后截图显示

#### 6. 字幕显示模块 (Subtitle)
```python
def display_subtitle(text, emotion="neutral"):
    """显示字幕和角色"""
    # 清屏
    os.system('clear')

    # 显示角色图片
    os.system(f'imgcat assets/waifu/{emotion}.gif')

    # 显示分隔线
    print("\n" + "="*50)
    print("💬 Coding Waifu 讲解")
    print("="*50 + "\n")

    # 显示字幕（支持打字机效果）
    for char in text:
        print(char, end='', flush=True)
        time.sleep(0.03)  # 打字机效果

    print("\n")
```

## 实现路线图

### Phase 1: 基础框架（1-2天）
- [x] 项目初始化
- [ ] 创建基础目录结构
- [ ] 实现简单的 iTerm2 输出显示
- [ ] 测试 imgcat 图片显示

### Phase 2: 监控和分析（2-3天）
- [ ] 实现 Claude Code 输出监控
- [ ] 解析关键事件（读文件、写文件、执行命令）
- [ ] 事件队列和处理逻辑

### Phase 3: 语音和显示（2-3天）
- [ ] 集成 TTS（先用 macOS say）
- [ ] 准备 waifu 角色素材
- [ ] 实现图片 + 字幕同步显示

### Phase 4: 智能讲解（3-4天）
- [ ] 设计讲解模板
- [ ] 集成 LLM 生成更自然的讲解
- [ ] 优化讲解时机和内容

### Phase 5: 优化和美化（2-3天）
- [ ] 添加更多表情和动画
- [ ] 优化 TTS 声音质量
- [ ] 添加配置文件
- [ ] 编写使用文档

## 技术栈选择

### 核心语言：Python 3.9+
**理由**：
- iTerm2 官方 Python API
- 丰富的 TTS/音频库
- 快速原型开发

### 关键依赖：
```python
# requirements.txt
iterm2                 # iTerm2 Python API
pyttsx3               # TTS（可选）
openai                # OpenAI API（TTS/GPT）
rich                  # 终端美化
click                 # CLI 工具
watchdog              # 文件监控
asyncio               # 异步处理
```

### 项目结构：
```
waifucoding/
├── src/
│   ├── monitor.py        # 监控 Claude Code
│   ├── analyzer.py       # 分析事件
│   ├── narrator.py       # 生成讲解
│   ├── tts.py           # 语音合成
│   ├── display.py       # 图片和字幕显示
│   └── main.py          # 主程序入口
├── assets/
│   ├── waifu/           # 角色图片/动画
│   └── sounds/          # 音效（可选）
├── config/
│   └── settings.yaml    # 配置文件
├── tests/
├── README.md
└── requirements.txt
```

## 配置示例

```yaml
# config/settings.yaml
waifu:
  name: "小AI"
  voice: "Ting-Ting"  # macOS 中文语音
  speed: 180          # 语速

monitor:
  method: "iterm2_api"  # 或 "file_watch", "tmux"
  poll_interval: 0.5    # 秒

narrator:
  mode: "template"      # 或 "llm"
  llm_model: "gpt-4o-mini"
  temperature: 0.7

display:
  avatar_size: "medium"  # small/medium/large
  subtitle_length: 100   # 字幕最大长度
  typewriter_effect: true
```

## 使用流程

1. **启动 iTerm2，开两个 tab**
   - Tab 1: 运行 Claude Code
   - Tab 2: 运行 `python main.py --monitor-tab=1`

2. **自动工作流程**：
   ```
   Claude Code 输出
        ↓
   Monitor 捕获
        ↓
   Analyzer 分析事件
        ↓
   Narrator 生成讲解
        ↓
   TTS 转语音 + Display 显示图片/字幕
        ↓
   Tab 2 播放呈现
   ```

3. **用户体验**：
   - 在 Tab 1 正常使用 Claude Code
   - Tab 2 自动显示 waifu 讲解员
   - 听语音讲解 + 看字幕 + 看角色动画

## 可选增强功能

1. **交互功能**：
   - 语音唤醒："小AI，解释一下这段代码"
   - 暂停/继续讲解
   - 调节语速

2. **历史记录**：
   - 保存讲解日志
   - 回放历史讲解

3. **多语言支持**：
   - 中英文自动切换
   - 多种声音选择

4. **情绪系统**：
   - 根据代码质量调整情绪
   - 发现 bug 时表现担心
   - 完成任务时庆祝

## 参考资源

- [iTerm2 Python API 文档](https://iterm2.com/python-api/)
- [iTerm2 图片显示文档](https://iterm2.com/documentation-images.html)
- [imgcat 工具](https://github.com/eddieantonio/imgcat)
- [OpenAI TTS API](https://platform.openai.com/docs/guides/text-to-speech)
