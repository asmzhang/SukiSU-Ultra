# SukiSU-Ultra (v4.0.0) Windows 编译指南

在 Windows 平台上编译 SukiSU-Ultra 需要配置底层的交叉编译工具链（NDK 及其 Sysroot），并解决 `bindgen` 对 C/C++ 头文件及 `libclang` 库的依赖问题。以下是完整配置和编译指令：

## 1. 准备环境

### 安装目标架构 (Rust)
安装用于交叉编译的 ARM64 目标架构：
```powershell
rustup target add aarch64-linux-android
```

### 安装完整 LLVM
`bindgen` 依赖完整的 `libclang.dll`。请确保不是使用精简版的 `llvm-mingw`。可以通过 winget 安装 LLVM 官方版：
```powershell
winget install LLVM.LLVM --location "C:\my\platform\LLVM"
```

## 2. 环境变量及 Cargo 设定

你需要让 Cargo 和 `bindgen` 指向同一个 NDK r29 和刚安装的 LLVM。
在终端执行以下 PowerShell 设定：

```powershell
# 1. 设置路径变量（需根据你本机的路径适配）
$ndkPath = "C:\my\platform\Android\Sdk\ndk\29.0.14206865"
$llvmPath = "$ndkPath\toolchains\llvm\prebuilt\windows-x86_64"
$ndkSysroot = "$llvmPath\sysroot"

# 2. 设置 libclang 的位置，供 bindgen 解析 C 头文件使用
$env:LIBCLANG_PATH = "C:\my\platform\LLVM\bin"

# 3. 设置 bindgen 寻找内建头文件的位置 (重点解决 linux/ioctl.h 找不到的问题)
$env:BINDGEN_EXTRA_CLANG_ARGS = "--sysroot=$ndkSysroot -I$ndkSysroot\usr\include\aarch64-linux-android"

# 4. 指定 Rust 交叉编译使用的 C/C++ 编译器和链接器
$env:CC_aarch64_linux_android = "$llvmPath\bin\aarch64-linux-android26-clang.cmd"
$env:CXX_aarch64_linux_android = "$llvmPath\bin\aarch64-linux-android26-clang++.cmd"
$env:AR_aarch64_linux_android = "$llvmPath\bin\llvm-ar.exe"
$env:CARGO_TARGET_AARCH64_LINUX_ANDROID_LINKER = "$llvmPath\bin\aarch64-linux-android26-clang.cmd"
```

## 3. 编译指令

### 编译后台守护程序 (Userspace `ksud` 等)

*(注意：在 `v4.0.0` 及以前版本中，[Cargo.toml](file:///f:/asmzhang/SukiSU-Ultra/Cargo.toml) 位于 `userspace/ksud` 而非项目根目录)*

```powershell
cd f:\asmzhang\SukiSU-Ultra\userspace\ksud
cargo clean --target aarch64-linux-android
cargo build --target aarch64-linux-android --release
```
**产物位置：**
`userspace\ksud\target\aarch64-linux-android\release\ksud`


### 编译管理端 APK (Manager)

```powershell
cd f:\asmzhang\SukiSU-Ultra\manager
.\gradlew.bat assembleRelease
```
**产物位置：**
`manager\app\build\outputs\apk\release\SukiSU_v4.x.x-release.apk`
