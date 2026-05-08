# TV 应用锁 (TV AppLock)

[![Android](https://img.shields.io/badge/Android-7.0%2B-brightgreen)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9-blue)](https://kotlinlang.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

一款专为 **Android TV / 电视盒子** 设计的应用锁。**免 Root，无需 ADB，不依赖悬浮窗权限**，仅通过无障碍服务 (AccessibilityService) 实现对任意应用的锁定保护。

手机也能用，但主要为遥控器操作优化。

---

## 功能

- **PIN 密码 / 图案密码** — 两种解锁方式，自由切换
- **任意应用锁定** — 从已安装应用列表中勾选，支持搜索
- **开机自启** — 设备重启后自动恢复保护
- **会话级解锁** — 解锁一次，离开应用即失效，再打开重验证
- **TV 遥控器优化** — D-Pad 导航、焦点缩放动画、全键盘快捷键
- **安全存储** — PBKDF2 + SHA-256 + 随机盐值加密，密码不可逆

## 为什么不用悬浮窗？

大部分应用锁使用 `TYPE_APPLICATION_OVERLAY` 或 `TYPE_ACCESSIBILITY_OVERLAY` 悬浮窗覆盖在被锁应用上。这个方案在部分设备上有已知缺陷：

- 点击穿透到底层应用（"按钮要按两下"）
- 悬浮窗状态机卡死导致无法退出，极端情况下会**锁死系统**
- MIUI 等定制 ROM 下悬浮窗触发的伪 accessibility event 会导致循环锁定

本项目采用 **Activity 覆盖方案**（`singleInstance` + 独立 `taskAffinity`），与 Norton AppLock 同款技术路线，安全可靠。

## 权限说明

| 权限 | 用途 | 是否必须 |
|------|------|----------|
| 无障碍服务 | 监听应用切换，检测被锁应用进入前台 | **必须** |
| 通知 (Android 13+) | 前台服务通知 | 建议 |
| 查询所有应用 | 读取已安装应用列表 | **必须** |

**不需要**：悬浮窗权限、Root、ADB、设备管理员。

## 编译

```bash
# 要求：JDK 17、Android SDK 34+

# 生成 Gradle Wrapper（首次）
gradle wrapper --gradle-version 8.13

# 编译 Debug APK
./gradlew assembleDebug

# APK 输出路径
# app/build/outputs/apk/debug/app-debug.apk
```

或直接用 Android Studio 打开项目 → Build → Build APK(s)。

## 技术架构

```
AccessibilityService (AppLockService)
  │  监听 TYPE_WINDOW_STATE_CHANGED
  │  isRealActivity() 过滤 MIUI 伪事件
  │  1.5s 防抖，避免重复锁屏
  │
  └── 检测到被锁应用进入前台
        │
        └── startActivity(LockScreenActivity)
              │  singleInstance, 独立 taskAffinity
              │  覆盖在被锁应用上方
              │
              ├── 输入正确 PIN/图案
              │     → broadcast APP_UNLOCKED
              │     → finish() 回到目标应用
              │
              └── 按返回/Home 键
                    → 强制回桌面，无法绕过
```

```
app/src/main/java/com/tv/applock/
├── MainActivity.kt                  # 主界面
│   ├── service/
│   │   ├── AppLockService.kt        # AccessibilityService 核心
│   │   └── BootReceiver.kt          # 开机自启
│   ├── ui/
│   │   ├── LockScreenActivity.kt    # PIN 锁屏
│   │   ├── PatternLockActivity.kt   # 图案锁屏
│   │   ├── PatternView.kt           # 图案绘制 View
│   │   ├── AppListActivity.kt       # 应用选择列表
│   │   ├── SetupActivity.kt         # 设置引导
│   │   └── SettingsActivity.kt      # 高级设置
│   ├── data/
│   │   ├── AppLockManager.kt        # 锁定列表管理
│   │   ├── PasswordManager.kt       # 密码存储/验证
│   │   └── InstalledAppsLoader.kt   # 已安装应用加载
│   └── util/
│       ├── CryptoUtils.kt           # PBKDF2 加密
│       └── TvFocusHelper.kt         # TV D-Pad 焦点辅助
```

## 下载

从 [Releases](../../releases) 页面下载最新 APK。

## License

MIT

---

# TV AppLock (English)

A root-free Android TV app locker. Lock any app behind a PIN or pattern — no root, no ADB, no overlay permission required. Uses AccessibilityService + Activity-based lock screen instead of WindowManager overlays for safety and reliability.

### Features

- PIN or pattern unlock
- Lock any installed app
- Auto-start on boot
- Session-based unlock (leave app = re-lock)
- TV remote (D-Pad) optimized
- PBKDF2 + SHA-256 password storage

### Quick Start

```bash
./gradlew assembleDebug
# APK: app/build/outputs/apk/debug/app-debug.apk
```

### Requirements

- Android 7.0+ (API 24)
- Enable Accessibility Service for "TV 应用锁" in system settings
- That's it. No other permissions needed.
