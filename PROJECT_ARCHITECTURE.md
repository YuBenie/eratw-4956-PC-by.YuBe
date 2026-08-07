# eraTW 4.981 项目架构文档

> 基于 Emuera_skia 引擎的东方同人 era 游戏
> 版本：4.981 (20260613) / 日语交流用画蛇添足版CH

---

## 目录

1. [项目概述](#1-项目概述)
2. [技术栈](#2-技术栈)
3. [仓库目录结构](#3-仓库目录结构)
4. [核心模块详解](#4-核心模块详解)
5. [分层架构](#5-分层架构)
6. [ERB 语法参考](#6-erb-语法参考)
7. [编码规约与命名约定](#7-编码规约与命名约定)
8. [本地化/翻译模式](#8-本地化翻译模式)
9. [魔改与 DLC 开发规范](#9-魔改与-dlc-开发规范)
10. [关键架构特征总结](#10-关键架构特征总结)

---

## 1. 项目概述

eraTW 是一款基于 **Emuera**（Era 游戏引擎的 C# 移植版本）的东方同人游戏。玩家在幻想乡中与各角色互动，进行日常指令、调教、战斗、探索等多种游戏内容。

本项目是中文社区魔改版本，包含大量 QOL（生活质量改进）、DLC 子系统、英文化翻译层等扩展。

运行方式：双击 `Emuera_skiaV8_x64.exe` 启动。

---

## 2. 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| **运行时引擎** | Emuera_skiaV8 | 基于 C# 的 Emuera 引擎变体，Skia 图形后端渲染 |
| **脚本语言** | ERB（EraBasic） | 日文风味的领域特定脚本语言（.ERB / .ERH） |
| **数据存储** | CSV | 所有变量常量定义、物品数据库、角色数据均以 CSV 管理 |
| **常量/头文件** | ERH | 预处理器常量定义和变量声明 |
| **UI 渲染** | HTML + Emuera 自定义标签 | ERB 中直接编写 HTML 按钮/div/shape 等，引擎负责渲染 |
| **音频** | SoundTouch_x64.dll | 音频处理库 |
| **辅助工具** | Python | `OPR_操作符字典_replace.py` 用于操作符字符串自动替换 |
| **插件体系** | .ERB 文件即插件 | 通过 DLC 子目录和开关变量实现热插拔式扩展 |
| **键盘宏** | macro.txt | 快捷键配置 |
| **懒加载** | lazyloading.cfg | 角色数据懒加载配置 |

---

## 3. 仓库目录结构

```
eratw4.981-260613-ENKI/
├── Emuera_skiaV8_x64.exe          # 游戏引擎主程序（双击启动）
├── icon.ico                       # 程序图标
├── .gitignore                     # Git 忽略规则
├── .nomedia                       # 媒体扫描忽略标记
├── macro.txt                      # 键盘宏快捷键配置
├── lazyloading.cfg                 # 角色数据懒加载配置
├── lazyloading_player_readme.txt  # 懒加载说明
├── OPR_操作符字典.csv              # 操作符/常量中日双语映射字典
├── OPR_操作符字典_replace.py      # 字典自动替换 Python 脚本
├── README.zh.md                   # 中文说明文档
│
├── CSV/                           # ★ 数据定义层 ★
│   ├── *.csv                      #   核心数据表
│   │   ├── Abl.csv                #   能力（話術技能、施虐属性...）
│   │   ├── Base.csv               #   基础属性（体力、気力、精力...）
│   │   ├── Talent.csv             #   特质（恋慕、思慕、大胃王...）
│   │   ├── Mark.csv               #   刻印（反発刻印...）
│   │   ├── Item.csv               #   物品（媚薬、バイブ...）
│   │   ├── CFLAG.csv              #   角色标志枚举
│   │   ├── TFLAG.csv              #   临时标志枚举
│   │   ├── FLAG.csv               #   系统标志枚举
│   │   ├── Equip.csv              #   装备枚举
│   │   ├── Stain.csv              #   污渍枚举
│   │   ├── source.csv             #   情绪源枚举（歓楽、受動...）
│   │   ├── Palam.csv              #   PALAM 值枚举
│   │   ├── TCVAR.csv              #   角色临时变量枚举
│   │   ├── Tequip.csv             #   临时装备枚举
│   │   ├── exp.csv                #   经验枚举
│   │   ├── Train.csv              #   训练指令名
│   │   ├── Str.csv                #   字符串常量
│   │   ├── DAY.csv / TIME.csv     #   日期/时间常量
│   │   ├── M_CFLAG.csv / M_FLAG.csv / M_TFLAG.csv  # 魔改作者标志
│   │   ├── BUFF.csv               #   Buff 枚举
│   │   ├── Juel.csv               #   宝珠枚举
│   │   ├── CSTR.csv               #   字符串常量
│   │   ├── GameBase.csv           #   游戏基础参数
│   │   ├── VariableSize.csv       #   变量尺寸配置
│   │   └── VarExtGameData.csv     #   扩展游戏数据
│   ├── *.als                      #   别名文件(名称→数字ID映射)
│   ├── _default.config            #   默认配置
│   ├── _fixed.config              #   固定配置
│   ├── _Rename.csv                #   重命名映射
│   ├── _Replace.csv               #   替换映射
│   ├── 字符串枚举.csv              #   字符串枚举表
│   └── Chara/                     #   角色 CSV 数据
│
├── ERB/                           # ★ 脚本逻辑层 ★
│   ├── *.ERB                      #   根级核心脚本
│   │   ├── SYSTEM.ERB             #     新游戏/读档处理
│   │   ├── COMMON.ERB             #     公共函数库（4447+ 行）
│   │   ├── COMMON_J.ERB           #     日文公共函数
│   │   ├── COMMON_TWG.ERB         #     TWG 公共函数
│   │   ├── COMMON_HTML.ERB        #     HTML 辅助函数
│   │   ├── SET_CMN.ERB            #     设置通用函数
│   │   ├── TITLE.ERB              #     标题画面
│   │   ├── BATTLE.ERB             #     战斗系统
│   │   ├── CHARA_LIST.ERB         #     角色列表
│   │   ├── NOUMIN.ERB / YASAI.ERB #     农耕/蔬菜系统
│   │   ├── CONDOM.ERB             #     安全套系统
│   │   ├── exp_bar.ERB            #     经验条
│   │   ├── VIDEO.ERB              #     视频/动画
│   │   ├── 初期設定.ERB           #     初期化设置
│   │   ├── 日時天候管理.ERB       #     日期/天气管理
│   │   ├── 時間停止解除.ERB       #     时停解除
│   │   ├── 天候*.ERB              #     天气系统
│   │   ├── 顔絵表示.ERB           #     脸部立绘显示
│   │   ├── アイテム解説.ERB       #     物品说明
│   │   ├── リソース作成.ERB       #     资源创建
│   │   └── リソース作成.ERB
│   ├── DIM.ERH                    #   全局常量定义（版本号、地点、颜色、位标记等）
│   ├── DIM_als.ERH                #   别名常量定义
│   ├── EJAC_EVENT.ERH             #   射精事件常量
│   ├── ANOTHER_TALK.ERB           #   额外对话
│   │
│   ├── Headers/                   #   自动生成的常量头文件
│   │   ├── AutoConst_Abl.ERH      #     能力常量
│   │   ├── AutoConst_Base.ERH     #     基础属性常量
│   │   ├── AutoConst_CFLAG.ERH    #     角色标志常量
│   │   ├── AutoConst_FLAG.ERH     #     系统标志常量
│   │   ├── AutoConst_TFLAG.ERH    #     临时标志常量
│   │   ├── AutoConst_Talent.ERH   #     特质常量
│   │   ├── AutoConst_Item.ERH     #     物品常量
│   │   └── ...（共 28 个 AutoConst_*.ERH）
│   │
│   ├── コマンド関連/              # ★ 指令系统（核心交互模块）★
│   │   ├── COMF/                  #     指令本体 (COMmand Files)
│   │   │   ├── COMF{数字} 指令名.ERB   # 基本调教指令 (0~200+)
│   │   │   ├── 日常系/            #     日常指令 (300~431)
│   │   │   │   ├── COMF300 会話.ERB    # 会话
│   │   │   │   ├── COMF414 食事.ERB    # 吃饭（906行）
│   │   │   │   └── COMF431 お風呂.ERB  # 洗澡放松
│   │   │   ├── 700 自慰系/        #     自慰指令
│   │   │   └── COMF270-299KOJO.ERB     # 自定义口上指令
│   │   ├── SCOMF/                 #     派生指令 (Sub-COMF)
│   │   ├── USERCOM_*.ERB          #     用户指令菜单处理
│   │   └── COMF_DEF.ERB           #     指令定义
│   │
│   ├── イベント関連/              # ★ 事件系统 ★
│   │   ├── EVENTTURNEND.ERB       #     回合结束处理
│   │   ├── AFTERTRA.ERB           #     调教后处理
│   │   ├── 妊娠関連/              #     妊娠/生子子系统
│   │   │   ├── PREGNANCY.ERB
│   │   │   ├── BIRTH.ERB
│   │   │   ├── CHILD_MOVEMENT.ERB
│   │   │   └── ...
│   │   └── ...
│   │
│   ├── ステータス表示関連/        #   状态显示系统
│   │   ├── INFO.ERB               #     角色信息
│   │   ├── Look.ERB               #     外表描述
│   │   ├── PRINT_STATE.ERB        #     状态打印
│   │   └── 能力表示.ERB           #     能力显示
│   │
│   ├── ステータス計算関連/        #   数值计算系统
│   │   └── SOURCE/                #     情绪源计算
│   │
│   ├── 口上・メッセージ関連/      # ★ 台词/消息系统 ★
│   │   ├── KOJO_MESSAGE.ERB       #     核心台词引擎
│   │   ├── COMMON_KOJO.ERB        #     公共口上
│   │   ├── EVENT_MESSAGE.ERB      #     事件消息
│   │   ├── EVENT_MESSAGE_COM*.ERB #     各指令域消息
│   │   ├── AUTO_AEGI.ERB          #     自动喘息
│   │   └── 個人口上/              #     各角色专属台词
│   │
│   ├── 魔改内容/                  #   社区魔改功能
│   │   └── qol/                   #     生活质量改进
│   │       ├── QOL_USERCOM.ERB    #        指令菜单增强
│   │       └── ...
│   │
│   ├── fromEN/                    # ★ 英文化翻译层 ★
│   │   ├── TR_REPLACEMENT.ERB     #     字符串替换库
│   │   ├── TR_VARS.ERH            #     翻译变量
│   │   ├── VARSET.ERH             #     变量集
│   │   ├── _TR Lib.ERB            #     翻译库函数
│   │   ├── BUGFIX.ERB             #     Bug 修复
│   │   ├── SEX_CONSTANTS.ERH      #     H 常量
│   │   ├── QOL/                   #     英化 QOL
│   │   │   ├── QOL_COM_HIGHLIGHTING.ERB  # 指令高亮
│   │   │   └── ...
│   │   ├── PREGNANCY/             #     妊娠系统英化
│   │   ├── LOCATIONS/             #     地点英化
│   │   ├── Custom_Commands/       #     自定义指令
│   │   ├── ModularCharacterUpdate.ERB/.ERH  # 模块化角色更新
│   │   ├── ANON/                  #     匿名化
│   │   └── ...（其他翻译覆盖模块）
│   │
│   ├── DLC/                       # ★ DLC 子系统（可选功能插件）★
│   │   ├── DLC.ERH                #     DLC 常量
│   │   ├── DLC FUNCTION.ERB       #     DLC 功能函数
│   │   ├── DLCEVENT.ERB           #     DLC 事件
│   │   ├── TRYCALLDLC.ERB         #     DLC 调用接口
│   │   ├── 魔法DLC/               #     魔法扩展
│   │   ├── 摄影系统.ERB           #     摄影
│   │   ├── 双重函数.ERB           #     双重 play
│   │   ├── 一键扫除.ERB           #     一键扫除
│   │   ├── 告白之神.ERB           #     告白之神
│   │   ├── 诱导推倒.ERB           #     诱导推倒
│   │   ├── 自機立绘补丁.ERB       #     自机立绘
│   │   ├── 大别墅.ERB             #     大别墅系统
│   │   ├── 开锁系统.ERB           #     开锁
│   │   ├── 自室描写.ERB           #     自室描写
│   │   ├── MY宴会.ERB             #     宴会系统
│   │   ├── 神签/                  #     神签系统
│   │   ├── 实用设置补丁/          #     实用设置
│   │   ├── 音乐补丁/              #     BGM 系统
│   │   ├── GENSOKYO_MAP_FIX/      #     地图修复
│   │   ├── MEDICINE/              #     药品
│   │   └── ...
│   │
│   ├── MOVEMENTS/                 #   移动/地图系统
│   │   └── JOB_仕事内容.ERB       #     工作内容
│   │
│   ├── NEWGAME/                   #   新游戏初始化
│   ├── BODY_INFO/                 #   身体信息/立绘
│   ├── CHARALIST_DT/              #   角色列表（分类/过滤/排序）
│   ├── COLOREDMAPS/               #   彩色地图系统
│   ├── CASINO/                    #   赌场小游戏
│   ├── 衣服/                      #   服装系统
│   ├── ビジュアル関連/            #   视觉/图像处理
│   ├── OBJ/                       #   对象/物品
│   ├── 潜伏モード関連/            #   潜行/潜伏模式
│   ├── グラフィック表示ライブラリ/ #   图形显示库
│   ├── method_from_eratohoЯeverse/ #   来自其他 era 的 method 函数
│   ├── SHOP関連/                  #   商店系统
│   ├── カラム機能/                #   列功能
│   ├── demo/                      #   测试/演示文件
│   └── キャラデータ/              #   角色数据
│
├── resources/                     # 引擎资源文件
├── sound/                         # 音频/音效资源
├── font/                           # 字体文件
│   └── [字体安装说明].txt
├── plugins/                       # 外部插件目录
├── tool/                          # 开发工具
├── bad/                           # bad_apple ASCII艺术作品（1153个txt文件）
├── README集/                      # 各类历史 readme 文档
│   ├── eraTW4.881~ 修正パッチ.txt
│   ├── era紅魔館Readme/
│   ├── 小鈴和克勞恩皮絲作者readme/
│   └── ...
├── 原版+前人整合等各种readme/      # 原始文档/口上模板教程
└── 魔改版更新记录文档/             # 社区魔改更新记录
```

---

## 4. 核心模块详解

### 4.1 数据定义模块（CSV + Headers）

所有游戏变量的数字 ID → 名称映射在 CSV 中定义。ERB 通过 Headers 中的常量引用。

**核心 CSV 表：**

| 文件 | 用途 | 示例条目 |
|------|------|---------|
| `Base.csv` | 基础属性 | `0,体力` `1,気力` `6,精力` |
| `Abl.csv` | 能力 | `0,親密` `1,従順` `12,話術技能` |
| `Talent.csv` | 特质 | `0,恋慕` `1,思慕` |
| `Mark.csv` | 刻印 | `0,反発刻印` |
| `Item.csv` | 物品 | `40,ローター` `41,媚薬` |
| `CFLAG.csv` | 角色标志 | `0,睡眠` `1,現在位置` |
| `TFLAG.csv` | 临时标志 | `100, 胜利与否` `192, 结果类型` |
| `Train.csv` | 调教指令名 | — |

**常量生成机制：**
- `Headers/AutoConst_*.ERH` 由工具根据 CSV 自动生成
- 例如 `AutoConst_Base.ERH` 会生成 `#DIM CONST BASE_体力 = 0`
- ERB 代码中使用 `BASE_体力` 而非数字 `0`，保证可读性

### 4.2 指令系统（コマンド関連）

这是游戏的核心交互系统，采用 **COM + 数字编号** 的命名规范。

**每个指令由以下标签组成：**

| 标签函数 | 作用 | 示例 |
|---------|------|------|
| `@COM{N}` | 指令主逻辑入口 | `@COM431` |
| `@COM_ABLE{N}` | 可用性判定（返回 0/1） | `@COM_ABLE431` |
| `@COM{N}_DISPLAY` | 动态指令名显示 | `@COM431_DISPLAY` |
| `@COM{N}_TAGS` | 指令分类标签（位掩码） | `@COM414_TAGS` |
| `@COM{N}_TOUCH(ARG)` | 接触/体位判定 | `@COM60_TOUCH(ARG)` |

**指令分组：**

| 编号范围 | 类型 | 目录 |
|---------|------|------|
| 0~99 | 基本爱抚/调教 | `COMF/` |
| 100~199 | SM/道具系 | `COMF/` |
| 200~269 | 辅助指令 | `COMF/` |
| 270~299 | 自定义口上指令 | `COMF/` |
| 300~399 | 日常系指令 | `COMF/日常系/` |
| 400~499 | 移动/约会 | `COMF/日常系/` |
| 500~599 | 派生指令 (SCOM) | `SCOMF/` |
| 700~799 | 自慰系 | `COMF/700 自慰系/` |

**指令执行流程:**
```
玩家选中指令 N
  → USERCOM 调用 COM_ABLE{N} 判断可用
  → 调用 @COM{N} 主逻辑
    → 调用 KOJO_MESSAGE_SEND() 触发台词
    → 计算 SOURCE 情绪源
    → 更新 TIME（时间流逝）
    → RETURN 结果
```

### 4.3 角色系统

- **MASTER** = 玩家角色编号
- **TARGET** = 当前交互目标角色编号（0 = 无目标）
- 角色属性系统基于多维数组索引：
  - `CFLAG:角色编号:属性编号` — 角色标志
  - `BASE:角色编号:属性编号` — 基础属性
  - `ABL:角色编号:属性编号` — 能力值
  - `TALENT:角色编号:属性编号` — 特质（0/1 或位掩码）
  - `MARK:角色编号:属性编号` — 刻印
  - `TCVAR:角色编号:属性编号` — 临时变量
  - `TEQUIP:角色编号:属性编号` — 临时装备
  - `EXP:角色编号:属性编号` — 经验值

- 角色数据存储在 `CSV/Chara/` 目录
- 懒加载通过 `lazyloading.cfg` 配置
- 角色列表功能在 `CHARALIST_DT/` 中实现

### 4.4 口上/台词系统（口上・メッセージ関連）

采用**事件总线**模式，核心逻辑与台词分离：

```
指令执行 → KOJO_MESSAGE_SEND(事件类型, ...) → 查找角色口上文件 → 显示台词
```

**事件类型：**
- `"COM"` — 指令执行
- `"SUCCESS"` / `"FAILURE"` — 成功/失败
- `"ORGASM"` / `"MARK"` — 高潮/刻印
- `"EVENT"` — 日常事件

**口上查找优先级：**
1. `個人口上/角色编号/` — 角色专属台词
2. `EVENT_MESSAGE_COM{编号}.ERB` — 指令域消息
3. 通用消息函数

### 4.5 事件系统（イベント関連）

- **`EVENTTURNEND.ERB`**：每日结束的核心回调
- **妊娠子系统**：`PREGNANCY.ERB` > `BIRTH.ERB` > `CHILD_MOVEMENT.ERB`
- **天气系统**：`日時天候管理.ERB` + `天候*.ERB / .ERH`
- **自由行动系统**：`FA16_HOLE` / `FA17_OBJ` 等变量驱动的角色自主行为

### 4.6 DLC/插件体系

DLC 是可选功能模块，通过变量开关在设置中启用/禁用：

```erb
IF 摄影系统开关 && 射影机
    CALL HTML_PRINTBUTTONC, @"摄像[770]", 770, BTN_COLOR
ENDIF
```

**现有 DLC：**
- 魔法系统 (`魔法DLC/`)
- 摄影系统 (`摄影系统.ERB`)
- 双重 Play (`双重函数.ERB`)
- 一键扫除 (`一键扫除.ERB`)
- 告白之神 (`告白之神.ERB`)
- 诱导推倒 (`诱导推倒.ERB`)
- 大别墅 (`大别墅.ERB`)
- 自室描写 (`自室描写.ERB`)
- 音乐/BGM 系统 (`音乐补丁/`)
- 神签系统 (`神签/`)
- 实用设置 (`实用设置补丁/`)
- 地图修复 (`GENSOKYO_MAP_FIX/`)

### 4.7 UI 渲染系统

使用 Emuera 特定的 HTML 扩展标签：

```erb
; 打印按钮
CALL HTML_PRINTBUTTONC, @"指令名[编号]", 编号, "颜色"

; 子菜单弹窗（QOL）
<button value='{选项ID}'>[选项名]</button>
<div rect='{X}px, {Y}px, {W}px, {H}px' border='...'>

; 形状/空位
<shape type='space' param='10px'>
<shape type='rect' param='...'>
```

**核心渲染函数：**
- `PRINTL` / `PRINTFORML` — 文本打印
- `HTML_PRINT` / `HTML_PRINTC` — HTML 渲染
- `HTML_PRINTBUTTONC` — 按钮渲染
- `INPUT` / `INPUT -1, 1` — 输入捕获（含鼠标）
- `CLIENTWIDTH()` / `CLIENTHEIGHT()` / `MOUSEX()` / `MOUSEY()` — 布局计算

### 4.8 英文化翻译层（fromEN）

翻译层通过**运行时字符串替换**实现，不修改原始文件：

- `TR_REPLACEMENT.ERB` — 主要替换函数
- `TR_VARS.ERH` — 翻译变量定义
- `VARSET.ERH` — 可修改变量集合
- `NAME_TR()` / `STR_TR()` — 名称/字符串翻译函数
- `ModularCharacterUpdate.ERB` — 模块化角色定义更新

**工作机制：**
```
原始代码 (日文): PRINTFORML %CALLNAME:TARGET%を犯す
翻译后: 日语引擎依然使用日文变量名
英文化层: 通过 TR 函数截取字符串显示时的名称
```

---

## 5. 分层架构

```
┌──────────────────────────────────────────────────────────────────┐
│                     表现层 (UI Layer)                            │
│  HTML 按钮/子菜单弹窗、脸部立绘(顔絵)、身体信息(BODY_INFO)、    │
│  图形渲染(グラフィック表示/ビジュアル)、彩色地图(COLOREDMAPS)、  │
│  BGM/音效(sound/)、键盘宏(macro.txt)                            │
│  [USERCOM_*.ERB] [QOL_USERCOM] [PLUGINS]                       │
├──────────────────────────────────────────────────────────────────┤
│                     应用逻辑层 (Application Layer)                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ 指令系统  │ │ 事件系统  │ │ 口上台词  │ │ 商店系统  │          │
│  │(COMF)    │ │(EVENT)   │ │(KOJO)    │ │(SHOP)    │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ 战斗系统  │ │ DLC插件   │ │ 赌场游戏  │ │ 农耕/采集 │          │
│  │(BATTLE)  │ │(DLC/)    │ │(CASINO)  │ │(NOUMIN)  │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
├──────────────────────────────────────────────────────────────────┤
│                     状态管理层 (State Layer)                      │
│  数值计算(SOURCE 情绪源系统)、恢复/伤害(REVOVER_PERMIL)、        │
│  妊娠/生子链路(PREGNANCY)、状态显示(INFO/PRINT_STATE)、          │
│  好感度/信赖度管理(TFLAG:信頼度管理)、Buff/BASE 增减             │
│  [ステータス計算関連/SOURCE] [ステータス表示関連] [妊娠関連]     │
├──────────────────────────────────────────────────────────────────┤
│                     数据访问层 (Data Layer)                       │
│  角色数据(CHARALIST_DT/キャラデータ)、物品(ITEM)、装备(EQUIP)、  │
│  服装(衣服)、移动/地图(MOVEMENTS/COLOREDMAPS)、                  │
│  身体信息(BODY_INFO)、角色列表(CHARA_LIST)                       │
│  [NEWGAME] [潜伏モード関連] [OBJ] [method_from_*]              │
├──────────────────────────────────────────────────────────────────┤
│                     基础设施层 (Infrastructure)                   │
│  COMMON.ERB(公共函数库)、SET_CMN(设置通用函数)、                │
│  SYSTEM.ERB(系统初始化)、TITLE.ERB(标题画面)、                   │
│  DIM.ERH(全局常量)、DIM_als.ERH(别名常量)、                     │
│  VARSET(变量集)、BUGFIX(修复)、QOL 工具函数                       │
│  [fromEN/lib] [method_from_eratohoЯeverse]                      │
├──────────────────────────────────────────────────────────────────┤
│                     翻译适配层 (Localization)                     │
│  fromEN: TR_REPLACEMENT(运行时替换), TR_VARS(翻译变量),         │
│  ModularCharacterUpdate(模块化角色), LOCATIONS(地点英化),       │
│  PREGNANCY/IMG/EN Grammar 等覆盖模块                             │
├──────────────────────────────────────────────────────────────────┤
│                     引擎层 (Engine)                               │
│  Emuera_skiaV8_x64.exe (C# + Skia 渲染)                         │
│  CSV 解析、ERB 解释执行、HTML 渲染、音频播放、存档系统           │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6. ERB 语法参考

### 6.1 注释

```
; 行注释（以分号开头）
```

### 6.2 标签/函数定义

```erb
@LABEL_NAME                     ; 无参数标签
@FUNC_NAME(ARG)                 ; 单参数函数
@FUNC_NAME(ARGS, ARG, ARG:1=0) ; 多参数 + 默认值
```

### 6.3 预处理器指令

```erb
#FUNCTIONS          ; 声明函数（自动 RETURN）
#FUNCTION           ; 单值返回函数
#LOCALSIZE N        ; 局部整数变量数量
#LOCALSSIZE N       ; 局部字符串变量数量
#DIM 变量名          ; 声明整数变量
#DIMS 变量名         ; 声明字符串变量
#DIM CONST 常量 = 值  ; 常量定义
#DIM DYNAMIC 变量名   ; 动态变量
#DIM SAVEDATA 变量名  ; 持久化存档变量
#DIM CHARADATA 变量名 ; 角色专属持久化变量
#DIM GLOBAL 变量名    ; 全局变量
#DEFINE 名称 值       ; 宏定义
[IF_DEBUG]           ; 调试模式条件编译
[ENDIF]
```

### 6.4 变量体系

#### 局部变量

```erb
LOCAL       ; 整数 (从 0 开始)
LOCAL:1     ; 一维数组
LOCAL:1:1   ; 二维数组
LOCALS      ; 字符串
LOCALS:1    ; 字符串数组
```

#### 全局/角色变量

| 前缀 | 含义 | 维度 | 示例 |
|------|------|------|------|
| `CFLAG:` | 角色标志（持久化） | `[角色:属性]` | `CFLAG:MASTER:現在位置` |
| `TFLAG:` | 临时标志 | `[属性]` | `TFLAG:193` |
| `FLAG:` | 系统标志 | `[属性]` | `FLAG:70` |
| `BASE:` | 基础数值 | `[角色:属性]` | `BASE:MASTER:体力` |
| `MAXBASE:` | 最大基础值 | `[角色:属性]` | `MAXBASE:ARG:気力` |
| `ABL:` | 能力值 | `[角色:属性]` | `ABL:MASTER:話術技能` |
| `TALENT:` | 特质 | `[角色:属性]` | `TALENT:恋慕` |
| `MARK:` | 刻印 | `[角色:属性]` | `MARK:反発刻印` |
| `TCVAR:` | 角色临时变量 | `[角色:属性]` | `TCVAR:TARGET:料理評価値` |
| `TEQUIP:` | 临时装备 | `[角色:属性]` | `TEQUIP:TARGET:体位` |
| `SOURCE:` | 情绪源 | `[角色:属性]` | `SOURCE:歓楽` |
| `EXP:` | 经验值 | `[角色:属性]` | `EXP:MASTER:会話経験` |
| `EQUIP:` | 装备 | `[角色:属性]` | `EQUIP:0` |
| `STAIN:` | 污渍 | `[角色:部位]` | `STAIN:MASTER:0` |
| `ITEM:` | 物品数量 | `[物品ID]` | `ITEM:簡易プール` |
| `PALAM:` | 快感参数 | `[角色:属性]` | `PALAM:CHARA:0` |
| `STR:` | 字符串数组 | `[索引]` | `STR:0` |
| `DOWNBASE:` | 降低基础值 | `[角色:属性]` | `DOWNBASE:気力` |
| `EX:` | 扩展数据 | `[角色:属性]` | `EX:MASTER:上次的入浴時间` |

#### 特殊变量

```erb
ARG          ; 函数第1整数参数
ARG:1        ; 第2整数参数（冒号后缀索引）
ARGS         ; 字符串参数
RESULT       ; 函数返回值（整数）
RESULTS      ; 函数返回字符串
TARGET       ; 当前目标角色编号
MASTER       ; 玩家角色编号
SELECTCOM    ; 当前选择指令编号
PREVCOM      ; 上一个执行指令编号
TIME         ; 当前分钟 (0~1440)
DAY          ; 当前天数
CHARANUM     ; 角色总数
PLAYER       ; 玩家角色（等同于 MASTER）
```

### 6.5 控制流

#### 条件语句

```erb
; 单行 IF
SIF 条件
    语句

; 多行 IF
IF 条件1
    语句
ELSEIF 条件2
    语句
ELSE
    语句
ENDIF
```

#### 运算符

```erb
=  +=  -=  *=  /=      ; 赋值
==  !=  >  <  >=  <=   ; 比较
&&  ||  !               ; 逻辑与/或/非
++  --                  ; 自增/自减
IS                      ; 特殊比较（用于 CASE IS < 400）
```

#### 选择语句

```erb
SELECTCASE 表达式
    CASE 值1, 值2        ; 多值匹配
        语句
    CASE 0 TO 5          ; 范围匹配
        语句
    CASE IS < 400        ; 条件匹配
        语句
    CASEELSE
        语句
ENDSELECT
```

#### 循环

```erb
FOR LOCAL, 0, 800       ; 变量, 起始, 上限(不含)
    语句
    CONTINUE             ; 跳过当前迭代
    BREAK                ; 跳出循环
NEXT
```

#### 跳转

```erb
GOTO LABEL
$LABEL
```

### 6.6 函数/METHOD 系统

```erb
; 调用
CALL 函数名              ; 无参数
CALL 函数名(参数1, 参数2)
CALLFORM COM_ABLE{LOCAL} ; 动态函数名

; 带异常捕获
TRYCALLFORM 函数名{}(参数)
TRYCCALLFORM 函数名(参数)
    [主逻辑]
CATCH
    [异常处理]
ENDCATCH

; 方法查询
GETMETH(@"COM{LOCAL}_DISPLAY", "Null")     ; 获取返回值
EXISTMETH(@"COM_HAS_OPTION{COM_ID}")       ; 检查是否存在

; 返回
RETURN 值       ; 返回整数（无清理）
RETURNF 值      ; 返回整数（自动执行 #FUNCTIONS 清理）
RETURN 0        ; 常规返回
RETURN 1        ; 成功返回
RETURN -1       ; 失败返回
```

### 6.7 内置函数

```erb
RAND:100                ; 随机 0~99
MAX(a, b)               ; 最大值
MIN(a, b)               ; 最小值
ABS(x)                  ; 绝对值
LIMIT(x, min, max)      ; 范围限制
INRANGE(x, min, max)    ; 范围检测 (含边界)
SQRT(x)                 ; 平方根

GETBIT(变量, 位)         ; 读位
SETBIT 变量, 位          ; 写位

GROUPMATCH(x, v1, v2, ...) ; 值集合匹配
STRCOUNT(str, substr)    ; 子串计数
STRLENS(str)             ; 字符串长度
STRCMP(str1, str2)       ; 字符串比较

GETCOLOR()               ; 当前前景色
GETBGCOLOR()             ; 当前背景色
MIX_COLOR(c1, r1, c2, r2) ; 颜色混合
HEX_TO_HTML_COLOR(c)    ; 十六进制→HTML颜色

CLIENTWIDTH()            ; 窗口宽度
CLIENTHEIGHT()           ; 窗口高度
MOUSEX() / MOUSEY()     ; 鼠标坐标
LINECOUNT()              ; 当前行数
HTML_STRINGLEN(str, mode) ; HTML字符串渲染宽度

GETTIME()                ; 获取系统时间戳
GET_TARGETNUM()          ; 当前目标数量
GET_MONTH()              ; 获取游戏内月份
CHECK_CHARA(id, type)    ; 检查角色类型
AT_HOME(char)            ; 是否在家
BATHCHECK(pos)           ; 检查是否有浴室
OPENPLACE(pos)           ; 检查是否开阔场地
```

### 6.8 字符串操作

```erb
'=                        ; 字符串赋值
LOCALS '= "hello"
STR:0 = %CALLNAME:TARGET% ; %...% 插值
@"文本{变量}更多文本"      ; 字符串格式化（@"" 语法）

NAME_TR(name_id)          ; 翻译名称
STR_TR(str_id)            ; 翻译字符串
TRAINNAME_TR(com_id)      ; 翻译指令名
```

### 6.9 显示/打印命令

```erb
PRINTL        ; 打印 + 换行
PRINTW        ; 打印 + 等待确认
PRINTPLAIN    ; 纯文本打印（无颜色）
PRINTS 变量    ; 打印字符串变量

PRINTFORML    ; 格式化 + 换行
PRINTFORMC    ; 格式化 + 颜色
PRINTC        ; 打印 + 颜色

PRINTBUTTON_EX("文本", 值, 启用状态)  ; 按钮
HTML_PRINT html, mode                 ; HTML 打印
HTML_PRINTC html                      ; HTML 居中打印
HTML_PRINTBUTTONC "文本", 值, "颜色"   ; HTML 按钮

SETCOLOR 颜色值                        ; 设置颜色
RESETCOLOR                             ; 重置颜色
DRAWLINE                              ; 画水平线
CLEARLINE n                           ; 清除 n 行
PRINTPLAIN                             ; 纯文本
```

### 6.10 常用 COLOR 常量

```erb
C_RED         = 0xFF0000
C_GREEN       = 0x00FF00
C_L_GREEN     = 0x90EE90
C_BLUE        = 0x0000FF
C_PINK        = 0xFFCCFF
C_HEARTPINK   = 0xFFC0CB
C_YELLOW      = 0xFFFF00
C_ORANGE      = 0xFFA500
C_GRAY        = 0x404040
C_L_GRAY      = 0x777777
C_PURPLE      = 0xCC99FF
C_WHITE       = 0xC0C0C0
C_BLACK       = 0x000000
```

---

## 7. 编码规约与命名约定

### 7.1 文件命名

| 文件类型 | 命名规则 | 示例 |
|---------|---------|------|
| 指令文件 | `COMF{数字} {名称}.ERB` | `COMF300 会話.ERB` |
| 派生指令 | `SCOMF{数字} {名称}.ERB` | `SCOMF1 シックスナイン.ERB` |
| 日常指令 | `COMF{数字} {名称}.ERB`（日常系目录下） | `COMF431 お風呂でくつろぐ.ERB` |
| 公共库 | `{功能名}.ERB` | `COMMON.ERB` |
| 常量头文件 | `AutoConst_{类别}.ERH` | `AutoConst_Talent.ERH` |
| 事件文件 | `EVENT_MESSAGE_{类型}.ERB` | `EVENT_MESSAGE_COM300.ERB` |
| DLC 文件 | `{DLC名}.ERB` | `摄影系统.ERB` |
| 翻译文件 | `{功能}_TR.ERB / TR_{类别}.ERB` | `TR_REPLACEMENT.ERB` |
| QOL 文件 | `QOL_{功能}.ERB` | `QOL_USERCOM.ERB` |

### 7.2 标签/函数命名

```
@COM{数字}               ; 指令主逻辑
@COM_ABLE{数字}          ; 指令可用性判定
@COM{数字}_DISPLAY       ; 指令显示名
@COM{数字}_TAGS          ; 指令标签
@COM{数字}_TOUCH(ARG)    ; 接触/体位判定

@COM{数字}_DEFINITION    ; 指令定义（极少数）
@COM_OPTION(COM_ID)      ; 子菜单处理
@COM_OPTION_ABLE{ID}(n)  ; 子选项可用性
@COM_NAME{ID}(n)         ; 子选项名称

@QOL_{功能名}             ; QOL 功能函数
@{模块名}_{功能名}        ; 模块化功能

CALL 関数名               ; 小驼峰/蛇形混合
```

### 7.3 变量命名

**局部变量：**
- 功能相关：`LOCAL`、`LOCAL:1`、`LOCALS`（默认临时变量）
- 业务命名：`回復前体力１`、`BathType`、`EAT_TIME`、`HIGHLIGHT_SCOM`
- 动态变量：`L_CURRENT_POSE`、`L_SCOM_NAME`（L_ 前缀表示局部）

**全局变量前缀模式：**
```erb
CFLAG:MASTER:現在位置     ; 中文命名（位置/状态类）
TFLAG:193                ; 数字标识（临时结果类）
ABL:MASTER:話術技能       ; 日文混合（能力名）
BASE:MASTER:体力          ; 中文命名（基础属性）
```

**变量名语言风格：**
- 中文：`体力`、`気力`、`料理評価値`、`現在位置`
- 日文：`話術技能`、`風呂`、`施虐属性`、`従順`
- 英文混入：`BathType`、`BONUS`、`MAX_ROWS`、`COL_WIDTH_PX`
- 数字下标：`回復前体力１`（表示"某角色的恢复前体力"）

### 7.4 代码组织风格

**分节注释（头部注释）：**
```erb
;-------------------------------------------------
;吃饭
;日常系指令、レベル1
;-------------------------------------------------
@COM414
```

**函数签名注释（COMMON.ERB 风格）：**
```erb
;-------------------------------------------------
;関数名:CHOICE
;概　要:２～４択関数
;引　数:ARGS:0…質問内容
;      :ARGS:1～4…選択肢の文字列(3,4は省略可)
;戻り値:ユーザ入力結果(0～3)
;-------------------------------------------------
```

**业务逻辑分节：**
```erb
;1 强制换行
;2 指令过滤
;3 获取文本属性
;4 生成按钮HTML
;5 制表控制
```

**代码块标签（GOTO 标签）：**
```erb
$INPUT_LOOP
$SKIP
$LOOP_END
```

**缩进风格：**
- 使用 Tab 缩进
- `SIF` 行：条件后缩进下一行
- `IF`/`ELSEIF`/`ELSE`/`ENDIF`：同一缩进层级
- `FOR`/`NEXT`：同一缩进层级

```erb
FOR LOCAL, 0, 10
	SIF 条件
		CONTINUE
	IF 条件
		代码
	ELSE
		代码
	ENDIF
NEXT
```

### 7.5 注释规范

```erb
; 普通注释
;; 重要注释/废止说明
;--- 分隔线 ---
;★ 功能标记
;TODO 待办
;FIXME 需修复
```

### 7.6 代码风格约定

1. **SIF 优先**：当只有一行需要条件执行时，使用 SIF 而非 IF...ENDIF
2. **RETURN 约定**：`RETURN 1` = 成功，`RETURN 0` = 正常退出，`RETURN -1` = 失败
3. **TFLAG:193 约定**：`1` = 大成功，`0` = 普通成功，`-1` = 失败，`-2` = 强制退出
4. **TFLAG:100 约定**：自动执行标记
5. **TFLAG:192 约定**：口上结果覆盖标记
6. **位掩码**：使用 `GETBIT`/`SETBIT` + `BIT_` 常量管理多个布尔状态
7. **字符串拼接**：使用 `'=` 操作符进行字符串连接
8. **数组初始化**：`VARSET LOCAL` 或 `VARSET 数组名` 清空变量

---

## 8. 本地化/翻译模式

### 8.1 架构

`fromEN/` 目录是英文化翻译层，采用**运行时替换**而非修改源文件：

```
原始代码（日文） → 引擎解析 → TR 函数截获 → 输出英文
```

### 8.2 核心机制

**`TR_REPLACEMENT.ERB`:**
- 定义字符串替换规则
- 在引擎调用 `NAME_TR()` / `STR_TR()` 时触发

**`TR_VARS.ERH`:**
- 定义翻译相关的变量和常量
- 与本地 `VARSET.ERH` 配合

**`ModularCharacterUpdate.ERB / .ERH`:**
- 模块化角色数据更新
- 允许在不改核心 CSV 的前提下添加新角色或修改属性

**关键函数：**
```erb
NAME_TR(id, mode)          ; 翻译名称（第2参数为翻译模式）
STR_TR(str_id)             ; 翻译字符串
TRAINNAME_TR(com_id)       ; 翻译指令名称
```

### 8.3 覆盖范围

| 模块 | 目录 |
|------|------|
| 公共字符串 | `fromEN/_TR Lib.ERB` |
| 地点名称 | `fromEN/LOCATIONS/` |
| 妊娠系统 | `fromEN/PREGNANCY/` |
| 图像资源 | `fromEN/IMG/` |
| QOL 功能 | `fromEN/QOL/` |
| 自定义指令 | `fromEN/Custom_Commands/` |
| 匿名化 | `fromEN/ANON/` |
| 新功能 | `fromEN/NEW_UPDATE/` |
| H 常量 | `fromEN/SEX_CONSTANTS.ERH` |
| 角色对话框 | `fromEN/CHARACTER_DIALOG_STATUS.ERB` |

### 8.4 操作符字典

`OPR_操作符字典.csv` 提供了**中文常量名 → 原始日文字符串**的映射。
`OPR_操作符字典_replace.py` 是一个 Python 脚本，用于自动将 ERB 文件中的日文字符串替换为常量名。

```csv
常量名,原文字符串
OPR_委托提示时,依頼提示時
OPR_成功,成功
OPR_失败,失敗
OPR_战斗前,戦闘前
```

---

## 9. 魔改与 DLC 开发规范

### 9.1 QOL 模式（生活质量改进）

QOL 功能位于：
- `ERB/魔改内容/qol/` — 中文魔改 QOL
- `ERB/fromEN/QOL/` — 英文 QOL

**设计原则：**
- QOL 函数覆盖/增强原始逻辑，不修改原文件
- 通过 `CALL QOL_SHOW_USERCOM` 替换原始 `USERCOM` 逻辑
- 使用 `GLOBAL_COMABLE()` / `TEMP_COM_DISPLAY_*` 等扩展接口

### 9.2 添加新指令

1. 在 `COMF/` 或 `COMF/日常系/` 创建 `COMF{编号} {名称}.ERB`
2. 定义以下标签：
   ```erb
   @COM{编号}_DISPLAY    ; 显示名
   @COM{编号}            ; 主逻辑
   @COM{编号}_TAGS       ; 标签
   @COM_ABLE{编号}       ; 可用性判定
   ```
3. 在 `USERCOM` 流中自动生效（QOL 版本遍历 0~800）

### 9.3 添加自定义指令

- 通过 `ADD_CUSTOM_COM` 和 `ADD_CUSTOM_SCOM` 接口
- `CALL Add_Custom_CALL_COM(TARGET)` 触发自定义指令处理
- `CALL QOL_IS_SCOM_CHECK` 检查派生指令可用性

### 9.4 DLC 开发约定

**开关模式：**
```erb
; 变量检查
IF DLC_{名称}开关 && 条件
    CALL DLC_{名称}_函数
ENDIF
```

**松耦合调用：**
```erb
; 使用 CALLFORM + 检查避免硬依赖
CALLFORM DLC_{名称}_函数(参数)
CATCH
    ; 未安装 DLC 时的降级处理
ENDCATCH
```

**添加 DLC 步骤：**
1. 在 `ERB/DLC/` 下创建子目录或文件
2. 定义开关变量（通常在 `DLC.ERH` 或 `DLC FUNCTION.ERB`）
3. 在 `DLCEVENT.ERB` 中注册事件挂载点
4. 通过 `TRYCALLDLC.ERB` 实现调用

### 9.5 位掩码标记约定

```erb
; 位置标记 (1p{N} 语法)
#DIM CONST 場所_風呂 = 1p2
#DIM CONST 場所_厨房 = 1p3

; COM 标签 (BIT_ 前缀)
#DIM CONST BIT_食事コマンド = 2
#DIM CONST BIT_風チラあり = 4
#DIM CONST BIT_家事コマンド = 17

; 自由行动标签
#DIM CONST BIT_自由行動風チラ禁止 = 1
```

### 9.6 异常安全

```erb
; 使用 TRYCCALLFORM 保护可能不存在的函数调用
TRYCCALLFORM CAN_SCOM{ARG}(1)
    RETURN RESULT
CATCH
    RETURN 1
ENDCATCH
```

### 9.7 操作符/常量命名

```erb
; OPR_ 前缀 = 操作符常量（中文字符串→日文字符串映射）
#DIMS CONST OPR_成功 = "成功"
#DIMS CONST OPR_失败 = "失敗"

; BIT_ 前缀 = 位标记
#DIM CONST BIT_V使用コマンド = 11

; C_ 前缀 = 颜色常量
#DIM CONST C_HEARTPINK = 0xFFC0CB

; COM 编号常量
#DIM CONST COM陰蒂夾 = 42
```

---

## 10. 关键架构特征总结

### 10.1 数据驱动
所有游戏属性（能力、特质、物品、标志、地点）在 CSV 中定义数字 ID，通过 AutoConst 头文件转为有意义的常量名。ERB 代码引用常量名而非硬编码数字，保证可维护性。

### 10.2 指令即入口
玩家每个操作对应一个 `@COM{编号}` 标签函数。引擎通过按数字编号调度，`USERCOM` 负责菜单渲染和可用性判断。

### 10.3 指令命名三件套
每个指令由三个核心标签组成：
- `@COM{N}` — 执行逻辑
- `@COM_ABLE{N}` — 可用性判定（返回 0/1）
- `@COM{N}_DISPLAY` — 动态显示名

### 10.4 口上（台词）与逻辑分离
角色台词通过 `KOJO_MESSAGE_SEND(事件类型, ...)` 事件总线发送，由角色专属口上文件处理，核心逻辑不直接依赖台词。

### 10.5 翻译即覆盖
`fromEN/` 翻译层通过 `TR_REPLACEMENT` 运行时替换字符串，不修改任何原始 ERB 文件，实现完全的非侵入式本地化。

### 10.6 松散耦合的 DLC
DLC 通过变量开关 + `CALLFORM`（动态调用，失败可捕获）实现可选挂载，不修改核心逻辑即可增加系统。

### 10.7 QOL 渐进增强
QOL 模块通过覆盖或增强原始函数（如替换 `SHOW_USERCOM` 为 `QOL_SHOW_USERCOM`）来改进用户体验，遵循"增强不破坏"原则。

### 10.8 位掩码风格
大量使用位操作（`GETBIT`/`SETBIT` + `1p{N}` 语法）在一个整数中存储多个布尔标志，用于地点特性、COM 分类、污渍类型等场景。

### 10.9 无数据库架构
所有游戏数据持久化通过 Emuera 存档系统（`#DIM SAVEDATA` / `#DIM CHARADATA` 标注的全局变量序列化），不依赖外部数据库。

### 10.10 中文社区魔改特性
本版本是中文社区魔改版本，包含日文原版基础上增加的：
- 中文指令名和操作符常量（`OPR_` 前缀）
- QOL 增强（子菜单弹窗、指令高亮、自定义 Shop 等）
- 大量 DLC 扩展
- 跨版本兼容层（`fromEN` + 魔改内容）

---

> 本文档基于 eraTW4.981 (20260613) 版本分析生成。
> 最后更新：2026-06-22