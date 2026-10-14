# 大麦抢票助手

在 Android 真机上，通过 adb 替你在**大麦 App** 里自动连点完成抢票：购买 → 确认 → 立即提交。走到支付宝收银台就停下，**付款永远由你自己完成**。

改写自 [prelearn-code/damai](https://github.com/prelearn-code/damai)「模式 0」，是一份 **REDMI K80 适配版**。

## ⚠️ 项目状态（2026-10-14，同一天同一台 K80 实战 6 场）

- ✅ **「不支持选座」的演出：已跑通。** 实测开售后 **5.3 秒**到达支付宝收银台，全程无风控、无异常弹窗。
- ⚠️ **「手动选座」的演出：已写代码，未实战。** 会停在选座页并点「确认选座」，但还没在真实开抢中验证过。
- ❌ **「一键下单 / 快速选座」的演出：未适配。** 这类是全新界面，脚本目前过不去。
- ⚠️ 有一个未修的 bug：确认页偶尔检测不到「立即提交」按钮。卡住超过 3 秒会自动保存截图，然后由你手动接手。

**结论**：抢「不支持选座」的演出，脚本能稳定从开售点跑到收银台。大麦有三种购票流程，界面完全不同，**抢之前先看详情页是哪一种**（见下方）。完整排查过程见 [`docs/实战复盘_2026-10-14.md`](docs/实战复盘_2026-10-14.md)。

## ⚖️ 使用前请知悉

- 自动化抢票**可能违反《大麦用户服务协议》**，可能导致账号被风控、限制购票甚至封号。实战中已多次触发大麦安全验证页。
- 本项目**不包含、也不接受**任何绕过风控、验证码、滑块、设备指纹的代码。遇到风控只返回一级，然后由你自己处理。
- 仅供学习 Android UI 自动化，**禁止用于倒卖、黄牛或任何商业用途**，使用后果自负。

## 支持的设备

| 设备 | 状态 |
| --- | --- |
| **REDMI K80**，屏幕分辨率设为 **WQHD+ 1440×3200** | ✅ 已实测（成功跑到收银台） |
| 其他 1440×3200 的安卓机 | ⚠️ 理论可用，需自己用预检和坐标检查确认 |
| 其他分辨率 | ❌ 需自己校准坐标，见 [适配其他手机](#适配其他手机) |
| iPhone / 鸿蒙 | ❌ 本版不支持 |

电脑端：**Windows 10/11**（bat、ps1 是 Windows 的；Python 部分跨平台）。

## 大麦的三种购票流程（抢之前先判断）

在演出**详情页**上看标注，就能知道脚本支不支持：

| 详情页标注 | 流程 | 脚本支持情况 |
| --- | --- | --- |
| 「**不支持选座**」 | 详情页 → 票档面板 → 确认页 → 支付宝 | ✅ 已跑通 |
| 「可选座」且**没有**「一键下单」 | 详情页 → 票档面板 → **选座页** → 确认页 → 支付宝 | ⚠️ 已写代码，未实战 |
| 有「**一键下单，智能随机分配座位**」 | 详情页 → 快速选座面板 → 确认页 → 支付宝 | ❌ 未适配（演出方开通，无法关闭） |

脚本**不会**帮你选场次、票价、观演人，这些必须在开售前在大麦里手动预选好。

## 安全规则（代码里写死的）

绝不点支付宝付款｜绝不点「返回重新选购」｜只认文字是「继续尝试」的按钮（不靠控件 ID）｜认不出的弹窗不点｜风控页只返回一级｜每次点击前确认大麦在前台｜屏幕分辨率必须完全一致，不一致直接退出。

---

## 第一次使用：配置

**1. 电脑装 Python 3.11+**：<https://www.python.org/downloads/>，安装时勾选 Add python.exe to PATH。

**2. 下载本项目**（不会 git 就用 GitHub 页面 Code → Download ZIP）：

```powershell
git clone https://github.com/ByteSparkX/damai.git
cd damai
```

**3. 建虚拟环境 + 装依赖**（环境名必须是 `.venv-phone`，两个 bat 按这个名字找）：

```powershell
python -m venv .venv-phone
.venv-phone\Scripts\python.exe -m pip install -r requirements-phone.txt
```

**4. 下载 adb**（Google 官方 platform-tools），解压到项目根目录：

```powershell
Invoke-WebRequest https://dl.google.com/android/repository/platform-tools-latest-windows.zip -OutFile pt.zip
Expand-Archive pt.zip -DestinationPath . -Force
Remove-Item pt.zip
```

## 手机设置（REDMI K80 / HyperOS）

| 步骤 | 路径 | 为什么 |
| --- | --- | --- |
| 开 USB 调试 | 设置 → 更多设置 → 开发者选项 → USB 调试 | 电脑才能连手机 |
| **开 USB 调试（安全设置）** | 同上页，需登录小米账号 | **不开的话点击会被系统静默拦截**，预检查不出这项 |
| **分辨率改 WQHD+** | 设置 → 显示与亮度 → 屏幕分辨率 → WQHD+ | 默认 FHD+ 坐标会不对 |
| 充电时保持唤醒 | 开发者选项 → 充电时屏幕保持唤醒 | 防息屏 |

插数据线，手机弹「允许 USB 调试」时勾选**始终允许**。验证：`platform-tools\adb.exe devices` 应看到一行 `xxxxxxxx  device`。

## 抢票前：在大麦里手动设好

1. 打开目标演出**详情页**；
2. 点「去设置」，**选好实名观演人**（几个人买几张）；
3. 点「预约抢票」，**选好场次和票价**并提交；
4. 详情页显示「已设置本次抢票信息 → 已完成」即可；
5. 支付宝里钱够用。

## 每次抢票

开售前把手机停在该演出**详情页**（不要停在票档面板），然后二选一：

**A. 双击 `grab_K80.bat`**，按提示输入开售时间（`YYYY-MM-DD HH:MM:SS`，按电脑本地时间）。它会先自动预检，有 FAIL 就不启动。

**B. 命令行**：

```powershell
.venv-phone\Scripts\python.exe scripts\damai_mode0_K80.py "2026-10-20 12:00:00"
```

开售前 2 秒开始连点，**这期间不要碰手机**。

**开抢时你要盯：**
- 跳到**支付宝收银台** → 脚本自动停，你核对金额后自己付款（订单一般只留十几分钟）；
- 显示 `terminal_no_retry_option` → 你选的票档卖完了，手动换票档或等回流票；
- 弹出滑块/验证码 → 脚本不处理，自己拖；
- 页面几秒不动 → 可能卡住，直接手动接手。想停脚本：`Get-Process python | Stop-Process。

---

## 看懂运行结果

结束时输出的 JSON，最重要是 **`status`**：

| `status` | 意思 | 你该做什么 |
| --- | --- | --- |
| `alipay_reached` | ✅ 到了支付宝收银台 | 核对后自己付款 |
| `terminal_no_retry_option` | 票档卖完了 | 手动换票档 / 等回流票 |
| `timeout` | 开售后 10 分钟没成功 | 看现场文件找原因 |
| `error` | 程序出错 | 看 `worker_error` 字段 |

跑的时候每行一条过程日志：`state_probe`（页面/状态变了）、`heartbeat`（页面没变，每秒报一次）、`dialog_snapshot`/`stuck_page_snapshot`（自动保存的现场）。

**自动保存的现场**（在 `artifacts\`）：卡住时脚本会保存截图 + 界面 XML，只读不点击。
- `stuck_retry_dialog_*`「继续尝试」卡住；`stuck_sold_out_dialog_*` 库存不足；`stuck_unknown_dialog_*` 认不出的弹窗；`stuck_order_page_loading_*` 等开售后在确认页卡住超 3 秒。

> 🔒 `artifacts\` 里会有你的手机号、观演人、订单信息，发 Issue 前务必打码；`.gitignore` 已排除整个目录。

## 工具

| 文件 | 作用 | 会点手机吗 |
| --- | --- | --- |
| `grab_K80.bat` | 预检 + 抢票 + 存结果 | ✅ |
| `precheck_K80.bat` / `tools/precheck_K80.py` | 只读预检 | ❌ |
| `scripts/damai_mode0_K80.py` | 抢票主程序 | ✅ |
| `scripts/damai_android_tail.py` | 确认页状态循环（原作者代码，未改） | — |
| `tools/inspect_point_K80.py` | 看点击坐标落在哪个控件 | ❌ |
| `tools/launch_at.ps1` | 定时启动 + 记录 + 导出日志 | ✅ |
| `tools/timeline_from_logcat.py` | 把手机日志整理成时间线，复盘用 | ❌ |
| `tools/test_k80_fix.py` | 26 个离线测试，不用手机 | ❌ |
| `docs/实战复盘_2026-10-14.md` | 6 场实战完整排查记录 | — |
| `docs/samples/*.xml` | 实战现场样本（已脱敏），测试用 | — |

## 适配其他手机

坐标按 1440×3200 校准。换手机：把 `scripts/damai_mode0_K80.py` 顶部的 `EXPECTED_SCREEN_SIZE` 改成你手机的实际分辨率，用 `tools\inspect_point_K80.py` 分别在详情页、票档面板确认 `MAIN_ACTION_POINT` 落在购买/确认按钮上。「继续尝试」「确认选座」的位置是实时读取的，不用校准。**改坐标时不要删掉分辨率校验。**

## 已知问题

- ❌ 确认页偶尔检测不到「立即提交」按钮，会空等。卡住超 3 秒会自动保存现场，拿到一份 `stuck_order_page_loading_*.xml` 就能定位。
- ❌ 「一键下单 / 快速选座」演出未适配。
- 🟡 大麦一更新，控件 ID / 页面结构可能失效。

## 参与修复

欢迎 Issue / PR。提 Issue 请附：手机型号、分辨率（`adb shell wm size`）、大麦版本、结果 JSON 的 `status` 和最后几十行、卡住时的 `stuck_*.png/.xml`（先打码）。提 PR 要求：跑通 `tools\test_k80_fix.py`、不破坏安全规则、**不接受**任何绕过风控/验证码/滑块/设备指纹或模拟真人操作躲避检测的代码。

## 致谢与版权

- 核心逻辑来自 **[prelearn-code/damai](https://github.com/prelearn-code/damai)**（模式 0），原作者文档保存在 `docs/upstream/`。
- 本仓库新增：K80 适配、按按钮文字识别弹窗、库存不足立即停止、实时读取按钮位置、选座页支持、卡住自动留证、预检/坐标检查/定时启动/时间线工具、离线测试。

> **关于许可证**：截至 2026-10-14，原仓库**没有声明开源许可证**，按默认著作权规则原作者保留所有权利。原作者代码的权利归原作者所有；本仓库新增代码可按需使用。若原作者有异议请提 Issue，会立即处理。