# Link Bubble Android APK 构建指南

## 项目概述

这是 Link Bubble（现在的 Brave Browser for Android）项目。该指南说明如何构建 APK 安装文件。

## 已完成的现代化更新

为了让这个 2015 年的项目能在现代 Android 系统上运行，已进行以下更新：

1. **仓库配置**: jcenter（已关闭）→ Google Maven + Maven Central
2. **Android Gradle Plugin**: 1.2.3 → 7.4.2
3. **Gradle**: 2.2.1 → 8.14.3（使用系统 gradle）
4. **编译 SDK**: 22 (Android 5.1) → 34 (Android 14)
5. **目标 SDK**: 22 → 34
6. **最低 SDK**: 16 → 21 (Android 5.0)
7. **Support Libraries**: 22.2.0 → 28.0.0（最后的 support library 版本）
8. **依赖声明**: compile → implementation
9. **移除**: Fabric/Crashlytics（已废弃）

## 前置要求

### 必需
- **JDK**: Java 11 或更高版本
- **Gradle**: 7.0 或更高版本（推荐 8.x）
- **Android SDK**: API Level 34
- **网络连接**: 首次构建需要下载依赖

### 可选
- **Android Studio**: 推荐使用 Android Studio 进行开发和构建
- **NDK**: 如果需要编译 native 代码（当前已禁用）

## 构建步骤

### 方法 1: 命令行构建 Debug APK（推荐快速测试）

```bash
# 1. 进入 Application 目录
cd Application

# 2. 使用 Gradle 构建 debug APK
gradle assemblePlaystoreDebug

# 或者使用 gradlew（如果网络可用）
./gradlew assemblePlaystoreDebug
```

**输出位置**:
```
Application/LinkBubble/build/outputs/apk/playstore/debug/LinkBubble-playstore-debug.apk
```

**应用 ID**: `com.linkbubble.playstore.dev`

### 方法 2: 命令行构建 Release APK（需要签名）

```bash
# 1. 创建构建脚本（从模板复制）
cp build-release.sh.template build-release.sh

# 2. 编辑 build-release.sh，设置以下环境变量：
#    - LINK_BUBBLE_KEYSTORE_LOCATION: keystore 文件路径
#    - LINK_BUBBLE_KEYSTORE_PASSWORD: keystore 密码
#    - LINK_BUBBLE_KEY_ALIAS: 密钥别名
#    - LINK_BUBBLE_KEY_PASSWORD: 密钥密码

# 3. 运行构建脚本
./build-release.sh
```

**输出位置**:
```
dist/LinkBubble-playstore-release.apk
```

**应用 ID**: `com.linkbubble.playstore`

### 方法 3: 使用 Android Studio

1. 启动 Android Studio
2. File → Open，选择 `Application` 目录
3. 等待 Gradle 同步完成（可能需要几分钟）
4. Build → Build Bundle(s) / APK(s) → Build APK(s)
5. 点击通知中的 "locate" 查看 APK 位置

### 方法 4: 直接安装到设备

```bash
# 构建并安装到连接的设备
cd Application
gradle installPlaystoreDebug

# 或者手动安装已构建的 APK
adb install -r LinkBubble/build/outputs/apk/playstore/debug/LinkBubble-playstore-debug.apk
```

## 常见问题

### 1. 网络问题 - 无法下载依赖

**错误**: `Could not GET 'https://...'`

**解决方案**:
- 确保网络连接正常
- 如果使用代理，配置 `~/.gradle/gradle.properties`:
  ```properties
  systemProp.http.proxyHost=proxy.example.com
  systemProp.http.proxyPort=8080
  systemProp.https.proxyHost=proxy.example.com
  systemProp.https.proxyPort=8080
  ```

### 2. Gradle 版本不兼容

**错误**: `Minimum supported Gradle version is X.X`

**解决方案**:
```bash
# 更新 gradle wrapper
gradle wrapper --gradle-version=8.14.3
```

### 3. SDK 未找到

**错误**: `SDK location not found`

**解决方案**:
创建 `Application/local.properties`:
```properties
sdk.dir=/path/to/android/sdk
```

### 4. 签名错误（Release 构建）

**错误**: `INSTALL_PARSE_FAILED_INCONSISTENT_CERTIFICATES`

**解决方案**:
```bash
# 卸载设备上已有的应用
adb uninstall com.linkbubble.playstore
```

### 5. 内存不足

**错误**: `OutOfMemoryError`

**解决方案**:
编辑 `Application/gradle.properties`:
```properties
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m
```

## 构建变体说明

当前项目有两个构建类型和一个产品风味：

### 构建类型
- **debug**: 开发版本，包含调试信息，应用 ID 添加 `.dev` 后缀
- **release**: 发布版本，启用代码混淆（ProGuard）

### 产品风味
- **playstore**: Google Play 商店版本

### 可用的构建变体
1. `playstoreDebug` - 测试版本（推荐）
2. `playstoreRelease` - 发布版本（需要签名）

## 构建命令总结

```bash
# 列出所有构建任务
gradle tasks

# 构建所有变体
gradle assemble

# 清理构建
gradle clean

# 构建并安装 debug 版本
gradle installPlaystoreDebug

# 仅构建 debug APK
gradle assemblePlaystoreDebug

# 构建 release APK（需要配置签名）
gradle assemblePlaystoreRelease

# 运行测试
gradle test
```

## 版本信息

- **应用版本**: 1.9.58
- **版本号**: 19580
- **包名**: com.linkbubble.playstore（release）/ com.linkbubble.playstore.dev（debug）

## 下一步建议

如果想让应用完全兼容最新的 Android 版本，建议进一步：

1. **迁移到 AndroidX**: 替换旧的 support library
2. **更新 targetSdk 到 35**: Android 15 适配
3. **更新第三方库**: Retrofit, Butterknife, Otto 等都有更新版本
4. **添加权限请求**: Android 6.0+ 需要运行时权限
5. **优化 UI**: 支持深色模式、适配刘海屏等
6. **性能优化**: 使用 R8 替代 ProGuard

## 配置文件说明

已从模板创建的配置文件：
- `Application/LinkBubble/fabric.properties` - Fabric API 配置（已废弃，保留以兼容）
- `Application/LinkBubble/src/main/java/com/linkbubble/ConfigAPIs.java` - API 密钥配置
- `Application/LinkBubble/src/main/AndroidManifest.xml` - 应用清单文件

## 需要的 API 密钥

如果要使用完整功能，需要配置：
- YouTube API Key（在 ConfigAPIs.java 中）
- Crashlytics/Fabric Key（已移除，可忽略）

## 参考资源

- [Android Developer Documentation](https://developer.android.com/)
- [Gradle Build Tool](https://gradle.org/)
- [Android Gradle Plugin Release Notes](https://developer.android.com/build/releases/gradle-plugin)
