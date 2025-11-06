# Link Bubble - Android APK 建置指南

## 專案概述

Link Bubble 是一個 2015 年的 Android 瀏覽器專案，已升級至現代 Android 開發標準以支援最新的 Android 版本。

## 目前配置

### 版本資訊

- **Gradle**: 8.7
- **Android Gradle Plugin**: 8.3.2
- **Compile SDK**: API 34 (Android 14)
- **Target SDK**: API 34 (Android 14)
- **Min SDK**: API 21 (Android 5.0)

### 主要升級內容

1. ✅ Gradle 2.2.1 → 8.7
2. ✅ Android Gradle Plugin 1.2.3 → 8.3.2
3. ✅ 遷移至 AndroidX
4. ✅ 移除已停用的 jcenter() 和 Fabric
5. ✅ 更新所有主要依賴套件

## 建置要求

### 必要工具

- JDK 17 或更高版本
- Android SDK (API 34)
- 穩定的網路連線（首次建置需下載約 500MB 依賴）

### 網路要求

建置過程需要訪問以下網域：

- `dl.google.com` - Google Maven Repository
- `repo.maven.apache.org` - Maven Central
- `services.gradle.org` - Gradle 分發下載
- `plugins.gradle.org` - Gradle 插件

## 建置步驟

### 1. Debug 版本（推薦測試用）

```bash
cd Application
./gradlew clean assembleDebug
```

**輸出位置**:
```
Application/LinkBubble/build/outputs/apk/debug/LinkBubble-debug.apk
```

### 2. Release 版本（需要簽名）

Release 版本需要設定以下環境變數：

```bash
export LINK_BUBBLE_KEYSTORE_LOCATION="/path/to/your/keystore.jks"
export LINK_BUBBLE_KEYSTORE_PASSWORD="your-keystore-password"
export LINK_BUBBLE_KEY_ALIAS="your-key-alias"
export LINK_BUBBLE_KEY_PASSWORD="your-key-password"
```

然後執行：

```bash
cd Application
./gradlew clean assemblePlaystoreRelease
```

**輸出位置**:
```
Application/LinkBubble/build/outputs/apk/playstore/release/LinkBubble-playstore-release.apk
```

### 3. 檢視所有可用任務

```bash
cd Application
./gradlew tasks --all
```

## 已知問題與解決方案

### DNS 解析問題

如果遇到 "Temporary failure in name resolution" 錯誤：

1. **檢查網路連線**：
   ```bash
   curl -I https://dl.google.com
   ```

2. **配置 DNS**：
   確保 `/etc/resolv.conf` 包含有效的 DNS 伺服器

3. **使用 Gradle 離線模式**（僅在已下載所有依賴後）：
   ```bash
   ./gradlew assembleDebug --offline
   ```

### 依賴下載緩慢

如果在中國大陸地區，可以配置使用阿里雲鏡像。編輯 `Application/build.gradle`：

```groovy
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/central' }
        google()
        mavenCentral()
    }
}
```

### 首次建置時間長

首次建置需要下載大量依賴，可能需要 10-30 分鐘，具體取決於網路速度。後續建置會使用緩存，速度會快很多。

## 預期編譯問題

由於這是從 Android API 22 升級到 API 34 的大版本跳躍，可能會遇到以下編譯問題：

### 1. 已棄用的 API

某些在 Android 5.1 中可用的 API 可能已被移除或改變：

- `GET_TASKS` 權限已被棄用
- `SYSTEM_ALERT_WINDOW` 需要特殊處理
- WebView 相關 API 可能有變更

### 2. AndroidX 遷移

雖然已啟用 Jetifier，但某些手動導入的 JAR 可能不相容：

- `picasso-2.1.1.jar` (2013年版本，可能需要更新)
- `jsoup-1.7.3.jar` (2013年版本，可能需要更新)

### 3. 運行時權限

Android 6.0+ 需要動態請求危險權限：

- `ACCESS_FINE_LOCATION`
- `WRITE_EXTERNAL_STORAGE`
- `READ_EXTERNAL_STORAGE`

### 4. 背景服務限制

Android 8.0+ 對背景服務有嚴格限制，可能需要：

- 遷移到 Foreground Service
- 使用 WorkManager 替代 Service

## 建置腳本範例

### 自動化建置腳本

創建 `build.sh`:

```bash
#!/bin/bash
set -e

echo "🔨 開始建置 Link Bubble APK..."

# 清理舊的建置
cd Application
./gradlew clean

# 建置 Debug 版本
./gradlew assembleDebug

# 檢查 APK 是否產生
APK_PATH="LinkBubble/build/outputs/apk/debug/LinkBubble-debug.apk"
if [ -f "$APK_PATH" ]; then
    echo "✅ 建置成功！"
    echo "📦 APK 位置: $APK_PATH"
    ls -lh "$APK_PATH"
else
    echo "❌ 建置失敗！未找到 APK 文件"
    exit 1
fi
```

## 安裝到裝置

### 使用 ADB

```bash
# 透過 USB 安裝
adb install -r Application/LinkBubble/build/outputs/apk/debug/LinkBubble-debug.apk

# 透過 Wi-Fi 安裝（需先配對裝置）
adb connect 192.168.1.100:5555
adb install -r Application/LinkBubble/build/outputs/apk/debug/LinkBubble-debug.apk
```

### 直接傳輸

將 APK 複製到手機後，直接點擊安裝。需要在設定中開啟「允許安裝未知來源的應用程式」。

## 疑難排解

### Gradle Daemon 問題

```bash
# 停止所有 Gradle Daemon
./gradlew --stop

# 清理 Gradle 緩存
rm -rf ~/.gradle/caches/
```

### 建置緩存問題

```bash
# 清理專案建置緩存
cd Application
./gradlew clean cleanBuildCache
```

### 記憶體不足

編輯 `Application/gradle.properties`，增加 JVM 記憶體：

```properties
org.gradle.jvmargs=-Xmx4096m -XX:MaxMetaspaceSize=1024m
```

## 開發建議

### 在 Android Studio 中開啟專案

1. 開啟 Android Studio
2. File → Open → 選擇 `Application` 目錄
3. 等待 Gradle 同步完成
4. 連接 Android 裝置或啟動模擬器
5. 點擊 Run 按鈕

### 程式碼現代化建議

1. **遷移到 Kotlin** - 提升程式碼安全性和簡潔性
2. **使用 Jetpack Compose** - 現代化 UI 開發
3. **實作 Material Design 3** - 改善使用者體驗
4. **使用 Hilt 進行依賴注入** - 替代當前的手動注入
5. **遷移到 Coroutines** - 替代舊的非同步處理方式

## 相關連結

- [Android Gradle Plugin 版本說明](https://developer.android.com/build/releases/gradle-plugin)
- [AndroidX 遷移指南](https://developer.android.com/jetpack/androidx/migrate)
- [Android API 變更](https://developer.android.com/about/versions)

---

**最後更新**: 2025-11-06
**維護者**: Claude Code
