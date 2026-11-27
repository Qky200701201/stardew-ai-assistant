# 🌾 Stardew AI Assistant — 星露谷物语 AI 游戏助手

通过内存读取实现星露谷物语的实时 AI 辅助工具，延迟仅 **1-5ms**（对比截图方案的 80-250ms）。

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Python](https://img.shields.io/badge/python-3.9+-blue.svg)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)

---

## ⚡ 快速导航

| 你的情况 | 看这里 |
|---------|--------|
| 从来没接触过 Cheat Engine | [📖 从零开始完整教程](#%E4%B8%80%E5%BF%AB%E9%80%9F%E5%BC%80%E5%A7%8B--3%E5%88%86%E9%92%9F%E8%BF%90%E8%A1%8C) |
| 会用 CE 但不知道找什么 | [🔍 Cheat Engine 详细操作步骤](#%E4%BA%8Ccheat-engine-%E8%AF%A6%E7%BB%86%E6%93%8D%E4%BD%9C%E6%AD%A5%E9%AA%A4) |
| 已经找到地址了 | [⚙️ 填入配置](#%E4%B8%89%E5%A1%AB%E5%85%A5%E9%85%8D%E7%BD%AE) |
| 想看代码原理 | [🧠 技术原理](#%E5%8A%9F%E8%83%BD%E8%AF%B4%E6%98%8E) |

---

## 一、快速开始（3 分钟运行）

### 前置条件

- ✅ Windows 系统
- ✅ 已安装 Python 3.9+（[下载地址](https://www.python.org/downloads/)）
- ✅ 已安装星露谷物语并运行正常
- ✅ 下载本项目的 [CE.exe](https://www.cheatengine.org/)（Cheat Engine，免费开源）

### 步骤概览

```
1. 启动星露谷物语 → 进入游戏任意场景
2. 打开 Cheat Engine → 选择 Stardew Valley.exe
3. 扫描金币数值 → 赚钱改变数值 → 缩小范围 → 找到地址
4. 把地址填入 reader.py
5. python hud.py → 看到悬浮窗！
```

详细步骤见下文 👇

---

## 二、Cheat Engine 详细操作步骤

### 2.1 安装 Cheat Engine

1. 访问 https://www.cheatengine.org/
2. 点击 Download → 选择对应版本（通常选 `cheatengine7.5.exe`）
3. 安装时如果弹出安全警告，选择"仍要运行"（CE 是开源安全的）

### 2.2 连接游戏进程

1. 启动星露谷物语，进入农场或任意场景
2. 打开 Cheat Engine
3. 点击左上角 **电脑图标** 🖥️
4. 在列表中找到 `Stardew Valley.exe`，选中它
5. 点击 **确定**

> 💡 如果找不到进程：确保游戏正在运行，且 CE 以管理员身份运行（右键 CE → 以管理员身份运行）

### 2.3 扫描金币 (Gold)

这是最简单的第一个目标，因为金币变化很明显。

**第一步：精确扫描**

1. Value Type → 选择 **4 Bytes**
2. Value Type → 选择 **Exact Value**
3. 输入你当前的金币数（比如 `500`）
4. 点击 **First Scan**
5. 此时下方会列出所有等于 500 的地址（可能有几百个）

**第二步：缩小范围**

1. 去矿洞打怪、摘星果、卖作物等方式赚取一些钱
2. 假设金币变成了 `510`
3. 在 CE 中点击 **New Scan**
4. Exact Value → 输入 `510`
5. 点击 **Next Scan**
6. 重复直到只剩 3-5 个地址

**第三步：确认正确地址**

1. 对每个剩余地址双击加入下方列表
2. 修改其中一个地址的值为 `9999`
3. 回到游戏查看金币是否真的变了
4. 如果变了 → 这就是正确的金币地址！记下它

### 2.4 获取指针链（关键步骤）

光有地址不够，游戏重启后地址会变。我们需要找到**指针链**。

1. 在下方列表中右键金币地址
2. 选择 **Find out what writes to this address**
3. 回到游戏赚一次钱（触发写入）
4. CE 会弹出一条汇编指令，类似：
   ```
   [stardewValley.exe+1A2B3C] - 8B 0D ? ? ? ? 8B 40 ?
   ```
5. 右键这条指令 → **Add address manually**
6. 在弹出的窗口中，你会看到类似：
   ```
   stardewValley.exe+00ABCDEF
   ```
7. 右键这个新地址 → **Browse memory view**
8. 在内存浏览器中，你会看到值被另一个地址引用
9. 继续这个过程，直到找到基址 `stardewValley.exe+XXXXXX`

**更简单的方法：自动查找指针**

1. 右键金币地址 → **Find out what addresses this pointer accesses**
2. 或者右键 → **Dissect allocated memory**
3. CE 会自动分析指针链

最终你会得到一个结构：
```
stardewValley.exe + 0xXXXXXX (基址)
    → + 0xXXXX (第一层偏移)
        → + 0xYYYY (第二层偏移)
            → = 金币值
```

### 2.5 扫描其他数据

用同样的方法扫描：

| 数据 | Value Type | 如何触发变化 |
|------|-----------|-------------|
| **血量** | 4 Bytes | 进矿洞被蝙蝠打一下 |
| **天数** | 4 Bytes | 等游戏内过一天（自然流逝） |
| **小时** | 4 Bytes | 等时间自然流逝 |
| **X坐标** | 4 Bytes | 走到不同位置 |
| **Y坐标** | 4 Bytes | 走到不同位置 |
| **季节** | 4 Bytes | 等季节转换（或通过 CE 搜索已知值） |
| **星期** | 4 Bytes | 等一周过去 |

> 💡 **小技巧**：对于不主动变化的数据（如季节），可以先在游戏设置里手动调整日期来扫描。

### 2.6 记录你的结果

完成扫描后，你应该有以下信息：

```
基址: stardewValley.exe + 0xXXXXXXXX
PlayerData 偏移: + 0xXXXX
Gold 偏移: + 0x40
Health 偏移: + 0x44
MaxHealth 偏移: + 0x48
Day 偏移: + 0xXXX
Hour 偏移: + 0xXXX
...
```

把这些值记下来，下一步要填到代码里。

---

## 三、填入配置

### 3.1 编辑 reader.py

打开 `reader.py` 文件，找到顶部的 CONFIG 区域（大约第 25-55 行）：

```python
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# CONFIG — 用 Cheat Engine 找到这些地址后填入
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

GAME_BASE_ADDRESS = 0x0000000000000000  # ← 替换为实际基址
PLAYER_OFFSET = 0x00000000              # ← PlayerData 相对基址的偏移
GOLD_OFFSET = 0x0000                    # ← 从 PlayerData 到 Gold 的偏移
HEALTH_OFFSET = 0x0000                  # ← 当前血量
MAX_HEALTH_OFFSET = 0x0000              # ← 最大血量
DAY_OFFSET = 0x0000                     # ← 当前天数
HOUR_OFFSET = 0x0000                    # ← 当前小时
MINUTE_OFFSET = 0x0000                  # ← 当前分钟
COORD_X_OFFSET = 0x0000                 # ← X 坐标
COORD_Y_OFFSET = 0x0000                 # ← Y 坐标
SEASON_OFFSET = 0x0000                  # ← 季节
DOW_OFFSET = 0x0000                     # ← 星期几
```

### 3.2 填写示例

假设你用 CE 找到：
- 基址: `0x00007FF6A2B00000`
- PlayerData: `基址 + 0x1A2B3C`
- Gold 在 PlayerData 内偏移 `0x40`
- Health 在 PlayerData 内偏移 `0x44`

那么填入：

```python
GAME_BASE_ADDRESS = 0x00007FF6A2B00000
PLAYER_OFFSET = 0x1A2B3C
GOLD_OFFSET = 0x40
HEALTH_OFFSET = 0x44
MAX_HEALTH_OFFSET = 0x48
DAY_OFFSET = 0x100
HOUR_OFFSET = 0x104
MINUTE_OFFSET = 0x108
COORD_X_OFFSET = 0x110
COORD_Y_OFFSET = 0x114
SEASON_OFFSET = 0x120
DOW_OFFSET = 0x124
```

> ⚠️ **注意**：上面的偏移量只是示例！你必须用自己的 CE 扫描结果替换。

### 3.3 验证配置是否正确

运行命令行模式测试：

```bash
python reader.py
```

如果输出类似这样，说明配置正确：

```
[20Hz] 金币:500 HP:20/20 第15天 14:30 春 📍农场 | ⚡low
```

如果全是初始值（金币0、HP0、第1天），说明地址没找对，需要重新用 CE 扫描。

---

## 四、运行方式

### 方式 A：命令行模式（纯数据）

```bash
python reader.py
```

适合调试和验证，每帧输出状态数据。

### 方式 B：GUI 悬浮窗模式（推荐）

```bash
python hud.py
```

会弹出一个半透明悬浮窗口，显示：
- 季节 / 日期 / 时间
- 金币数量
- 血量条（带颜色变化）
- 当前位置
- 紧急程度指示灯（绿→黄→红）
- 警告和建议

### 方式 C：仅运行分析器

```python
from analyzer import analyze_game_state, GameContext

ctx = GameContext(
    health_pct=45.0,
    gold=1250,
    season="夏",
    day=10,
    hour=18,
    location="森林",
)

result = analyze_game_state(ctx, mode="rule")
print(result)
```

---

## 五、功能说明

### 🔍 内存读取 (`reader.py`)

- 使用 Windows API `ReadProcessMemory` 直接读取游戏进程内存
- 延迟仅 1-5ms（对比截图方案 80-250ms）
- 支持静态基址和动态指针链两种方式
- 自动检测游戏进程

### 🧠 AI 分析 (`analyzer.py`)

**规则引擎模式（默认，零延迟）**：
- 血量危险警告（低于 20% 红色警报）
- 时间管理提醒（太晚回家防晕倒）
- 经济建议（金币多少的不同策略）
- 地点相关提示（矿井/农场/海边）
- 库存检查（种子/炸弹/食物）
- 季节转换提醒

**LLM 模式（可选，需 API Key）**：
- 接入 OpenAI GPT-4o-mini / Claude / 本地模型
- 更智能的情境分析和对话式建议
- 支持 NPC 好感度追踪

### 🎨 HUD 悬浮窗 (`hud.py`)

- 始终置顶的半透明窗口
- 实时血量条（绿→橙→红渐变）
- 紧急程度三色指示灯
- 警告和建议分区域显示
- ~10 FPS 刷新率

---

## 六、常见问题

### Q1: CE 找不到游戏进程？

- 确保游戏正在前台运行
- 以管理员身份运行 CE
- 试试 64-bit 和 32-bit CE 版本切换

### Q2: 读取到的值全是 0？

- 检查 GAME_BASE_ADDRESS 是否正确
- 确认游戏是否使用了 ASLR（每次重启基址会变）
- 改用指针链方式（设置 `USE_STATIC_BASE = False`）

### Q3: 游戏更新后地址变了怎么办？

- 重新用 CE 扫描即可，步骤和二一样
- 建议保存 CE 的 `.cetxt` 内存表文件方便下次导入

### Q4: 能用于联机游戏吗？

- ❌ **不建议**。联机游戏通常有反作弊系统
- 本项目仅适用于无反作弊的单机游戏
- 内存读取可能被封号

### Q5: 支持其他游戏吗？

- 理论上支持任何 PC 游戏
- 只需要针对每个游戏重新用 CE 扫描地址
- 核心代码不需要改动

### Q6: Python 报错 "ModuleNotFoundError"？

- 本项目只使用标准库（ctypes, tkinter, json）
- 不需要安装任何第三方库
- 确保使用 Python 3.9+

---

## 七、技术原理

### 为什么内存读取比截图快？

```
截图方案:
游戏画面 → GPU渲染 → 截屏(~30ms) → 传输(~20ms) → OCR/视觉模型(~100ms) → 语义理解
总延迟: ~150-250ms

内存读取:
游戏内存 → ReadProcessMemory(~0.5ms) → 直接读整数(~0ms)
总延迟: ~1-5ms
```

截图方案需要把像素变成语义（OCR 或 AI 视觉），而内存读取直接拿到结构化数值。

### 架构

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────┐
│  Stardew    │────▶│  Reader      │────▶│  Analyzer   │────▶│   HUD    │
│  Valley     │     │  (1-5ms)     │     │  (规则/LLM) │     │  (浮窗)  │
│  游戏进程    │     │  ReadProcMem │     │  JSON→建议   │     │  UI展示   │
└─────────────┘     └──────────────┘     └─────────────┘     └──────────┘
```

---

## 八、扩展开发

### 添加对新游戏的支持

1. 复制 `reader.py` 为 `newgame-reader.py`
2. 用 CE 扫描新游戏的内存地址
3. 修改 `GameState` 数据结构匹配新游戏
4. 修改 `analyze_state()` 中的规则

### 接入 LLM 深度分析

```python
from analyzer import LLMAnalyzer

llm = LLMAnalyzer(
    api_provider="openai",      # openai / anthropic / local
    api_key="sk-xxx",           # 或设置环境变量 OPENAI_API_KEY
    model="gpt-4o-mini",
)

# 从 reader 获取状态后
analysis = llm.analyze(game_context)
```

### 添加语音播报

```python
import edge_tts

async def speak(text):
    await edge_tts.Communicate(text, "zh-CN-YunxiNeural").save("temp.mp3")
    # 播放 temp.mp3
```

---

## 九、免责声明

本项目仅供学习和技术研究使用。

- 仅适用于您拥有合法授权的反作弊单机游戏
- 作者不对因使用本项目导致的任何后果负责
- 请勿在联机游戏或多玩家环境中使用
- 遵守当地法律法规和游戏服务条款

---

## 十、许可证

MIT License — 自由使用、修改、分发

---

*Made with ❤️ for the Stardew Valley community*
