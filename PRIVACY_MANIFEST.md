# OpenSSL Privacy Manifest 指南

## 概述

根据 Apple 的最新要求，所有 third-party SDK 和 XCFramework 都必须包含 Privacy Manifest 文件 (`PrivacyInfo.xcprivacy`)。本项目已经为 OpenSSL framework 自动添加了符合 App Store 要求的 Privacy Manifest 支持。

## 🔐 Privacy Manifest 内容

### OpenSSL 隐私声明

- **NSPrivacyTracking**: `false` - OpenSSL 不跟踪用户
- **NSPrivacyTrackingDomains**: `[]` - 无跟踪域名
- **NSPrivacyCollectedDataTypes**: `[]` - OpenSSL 本身不收集用户数据
- **NSPrivacyAccessedAPITypes**: 包含以下必需原因 API：

#### 已声明的必需原因 API

1. **NSPrivacyAccessedAPICategoryFileTimestamp** (原因: `0A2A.1`)
   - 用途：第三方 SDK 包装器，用于证书验证和文件操作
   - 涉及的 API：`stat`, `fstat`, `lstat` 等文件时间戳相关函数

2. **NSPrivacyAccessedAPICategorySystemBootTime** (原因: `35F9.1`)
   - 用途：设备上的时间测量，用于熵收集和随机数生成
   - 涉及的 API：`mach_absolute_time`, `systemUptime` 等

## 🚀 使用方法

### 自动集成（推荐）

当您运行 `./create-openssl-framework.sh` 时，Privacy Manifest 会自动添加到生成的 framework 中：

```bash
# 首先构建 OpenSSL 库
./build-libssl.sh --version=3.4.1

# 创建包含 Privacy Manifest 的 framework
./create-openssl-framework.sh
```

### 验证 Privacy Manifest

构建完成后，您可以验证 Privacy Manifest 是否正确添加：

```bash
# 检查单个 framework
ls -la frameworks/iPhoneOS/openssl.framework/PrivacyInfo.xcprivacy

# 检查 XCFramework
find frameworks/openssl.xcframework -name "PrivacyInfo.xcprivacy" -exec echo "Found: {}" \;
```

## 📋 自定义 Privacy Manifest

### 修改默认模板

如果您需要自定义 OpenSSL 的隐私声明，可以编辑模板文件：

```bash
# 编辑 OpenSSL 专用模板
vi app_privacy_manifest_fixer/Templates/UserTemplates/openssl.xcprivacy
```

### 添加额外的必需原因 API

如果您的应用使用了 OpenSSL 的其他功能，可能需要声明额外的 API：

```xml
<!-- 例如：如果使用了磁盘空间检查 -->
<dict>
    <key>NSPrivacyAccessedAPIType</key>
    <string>NSPrivacyAccessedAPICategoryDiskSpace</string>
    <key>NSPrivacyAccessedAPITypeReasons</key>
    <array>
        <string>E174.1</string>
    </array>
</dict>
```

## 🛠️ 高级用法

### 使用 app_privacy_manifest_fixer 工具

项目中包含了完整的 `app_privacy_manifest_fixer` 工具，可以自动分析和修复隐私清单：

```bash
# 安装工具到您的 iOS 项目
cd app_privacy_manifest_fixer
sh install.sh /path/to/your/ios/project

# 仅在 Archive 构建时运行（推荐）
sh install.sh /path/to/your/ios/project --install-builds-only
```

### 生成隐私访问报告

```bash
# 在 Xcode 中生成隐私报告
# Product -> Archive -> Generate Privacy Report
```

## 📚 相关文档

- [Apple Privacy Manifest Files](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files)
- [Required Reason API](https://developer.apple.com/documentation/bundleresources/describing-use-of-required-reason-api)
- [App Privacy Details](https://developer.apple.com/app-store/app-privacy-details/)

## ⚠️ 注意事项

1. **App Store 合规性**: 确保您的应用在 App Store Connect 中的隐私详情与 Privacy Manifest 一致
2. **定期更新**: 当 OpenSSL 版本更新或 Apple 要求变更时，可能需要更新 Privacy Manifest
3. **测试验证**: 在提交 App Store 前，请使用 Xcode 的 Privacy Report 功能验证所有隐私声明

## 🔧 故障排除

### 常见问题

**Q: 为什么我的 framework 中没有 PrivacyInfo.xcprivacy 文件？**
A: 请确保：
- 运行了 `create-openssl-framework.sh` 脚本
- `app_privacy_manifest_fixer/Templates/UserTemplates/openssl.xcprivacy` 文件存在
- 检查构建日志中的错误信息

**Q: App Store 拒绝了我的应用，说缺少必需原因 API 声明？**
A: 请检查：
- 您的应用是否使用了 OpenSSL 的其他功能
- 是否需要在 Privacy Manifest 中添加额外的 API 声明
- 使用 `app_privacy_manifest_fixer` 工具分析 API 使用情况

**Q: 如何为特定用途自定义 Privacy Manifest？**
A: 编辑 `app_privacy_manifest_fixer/Templates/UserTemplates/openssl.xcprivacy` 文件，添加或修改必要的声明。

## 📞 支持

如果您遇到任何问题或需要帮助，请：
1. 查看 [Apple 官方文档](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files)
2. 检查项目的 GitHub Issues
3. 使用 `app_privacy_manifest_fixer` 工具的分析功能 