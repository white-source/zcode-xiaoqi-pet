# zcode-xiaoqi-pet

小栖桌宠：一个 ZCode 插件 + 桌面宠物程序。小栖漂浮在所有窗口之上，实时展示 ZCode 多会话状态（思考 / 执行工具 / 等待确认 / 完成），带任务看板、彩色抽屉卡、通知、今日用量 / 套餐余量胶囊，还能让它说话。

移植自 [dsh-xiaoqi-pet](https://github.com/white-source/dsh-xiaoqi-pet)（DeepSeek Harness 版）：HTTP bridge 换成 ZCode 插件 hooks + 本地事件文件，不再需要端口和 token。

本仓库同时是一个 ZCode 插件市场的根目录：`marketplace.json` + `xiaoqi-pet/`（插件本体）+ `xiaoqi-pet/desktop/`（macOS 原生桌宠）+ `xiaoqi-pet/desktop/electron/`（Windows / Linux / macOS 通用桌宠）。

## 安装（给使用者）

**1. 装插件**：ZCode Desktop → **插件市场 → 添加 → 添加插件市场**，填入

```
white-source/zcode-xiaoqi-pet
```

然后在「小栖桌宠 / Xiaoqi Desk Pet」上点安装。hooks、skill 和 `/pet-say` 命令立即生效。

**2. 装桌宠本体**：到 [Releases](https://github.com/white-source/zcode-xiaoqi-pet/releases/latest) 下载对应平台：

| 平台 | 文件 | 说明 |
| --- | --- | --- |
| macOS 13+（Apple Silicon） | `ZCode-Xiaoqi-Pet-*-macos-arm64.zip` | 未签名：首次右键 → 打开 |
| Windows 10+ | `ZCode-Xiaoqi-Pet-*-win-x64.zip` / `-win-arm64.zip` | 解压运行 exe |
| Linux | `ZCode-Xiaoqi-Pet-*.AppImage` | `chmod +x` 后运行，需合成窗口管理器 |

**3. 试用**：跑任意一个 ZCode 会话，一秒内小栖就会醒来；`/pet-say 你好` 可以让它说话。

详细说明见 [xiaoqi-pet/README.zh-CN.md](xiaoqi-pet/README.zh-CN.md)（[English](xiaoqi-pet/README.md)）。

## 本地开发

```sh
# 插件校验（ZCode Desktop 自带的运行时可当 CLI）
node /Applications/ZCode.app/Contents/Resources/glm/zcode.cjs plugins validate xiaoqi-pet

# macOS 原生桌宠：编译并安装到 ~/Applications
xiaoqi-pet/desktop/build.sh --install

# Windows / Linux Electron 桌宠
cd xiaoqi-pet/desktop/electron && npm install && npm start

# 状态诊断（事件目录、会话状态、今日用量）
"$HOME/Applications/ZCode Xiaoqi Pet.app/Contents/MacOS/XiaoqiPet" --diagnose
```

本地调试市场时把仓库根目录直接添加为市场来源即可（`marketplace.json` 里的插件来源指向 GitHub，首次安装需要网络）。

## 发布

插件版本在 `xiaoqi-pet/.zcode-plugin/plugin.json`，市场条目在 `marketplace.json`，Electron 版在 `xiaoqi-pet/desktop/electron/package.json`，三者需保持一致。

1. 本地构建三平台产物：

```sh
# macOS（Apple Silicon）
xiaoqi-pet/desktop/build.sh
ditto -c -k --sequesterRsrc --keepParent \
  "xiaoqi-pet/desktop/dist/ZCode Xiaoqi Pet.app" \
  "ZCode-Xiaoqi-Pet-v0.3.0-macos-arm64.zip"

# Windows / Linux（在 macOS 或 Linux 上交叉构建均可）
cd xiaoqi-pet/desktop/electron && npm install
npx electron-builder --win zip --x64 --arm64 --publish never
npx electron-builder --linux AppImage --x64 --publish never
```

2. 建 tag 并上传 Release：

```sh
gh release create v0.3.0 --target main --title "小栖桌宠 v0.3.0" --notes "…" \
  "ZCode-Xiaoqi-Pet-v0.3.0-macos-arm64.zip" \
  "xiaoqi-pet/desktop/electron/dist/ZCode-Xiaoqi-Pet-0.3.0-win-x64.zip" \
  "xiaoqi-pet/desktop/electron/dist/ZCode-Xiaoqi-Pet-0.3.0-win-arm64.zip" \
  "xiaoqi-pet/desktop/electron/dist/ZCode-Xiaoqi-Pet-0.3.0-linux-x86_64.AppImage"
```

3. 更新 `marketplace.json` 与插件 manifest 的版本，让已安装的用户能收到更新。

## 与 DSH 版共存

macOS 上两者机器标识不同，可同时安装：ZCode 版是 `ZCode Xiaoqi Pet.app` / `com.xiaoqi.zcode-deskpet`，DSH 版是 `Xiaoqi Pet.app` / `com.xiaoqi.deskpet`。状态目录共享父目录（`XiaoqiPet/`）但文件互不干扰：DSH 用 `bridge.json`，ZCode 用 `feed/`、`emote/`、`balance.json`。

## 许可

MIT，见 [LICENSE](LICENSE)。形象与界面结构来自 dsh-xiaoqi-pet（MIT）。
