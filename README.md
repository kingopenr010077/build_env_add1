# build_env_add1

Android 14 离线构建环境补充包 — 提供 JDK 17、Gradle 8.4 及 Maven 本地仓库，配合 `build_env_base` 使用，用于在无网络环境中完成 AOSP/Android 项目的完整离线编译。

---

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `jdk-17.tar.gz` | 134 MB | OpenJDK 17 二进制包 |
| `gradle-8.4.tar.gz` | 250 MB | Gradle 8.4 完整发行版 |
| `maven-repo.tar.gz` | 734 MB | 预缓存 Maven 依赖仓库（AOSP 项目所需） |

**合计大小：约 1.1 GB（使用 Git LFS 存储）**

---

## 前置条件

在执行本仓库的安装步骤之前，**必须先克隆并配置好 `build_env_base`**：

| 仓库 | 提供内容 |
|------|----------|
| [build_env_base](https://github.com/your-org/build_env_base) | Android SDK、Gradle 缓存等基础环境 |

> `build_env_add1` 是 `build_env_base` 的补充包，二者缺一不可。

---

## 完整搭建步骤

### 1. 克隆两个仓库

```bash
# 克隆基础环境
git lfs clone https://github.com/your-org/build_env_base.git
cd build_env_base

# 克隆补充包
git lfs clone https://github.com/your-org/build_env_add1.git
cd build_env_add1
```

> 若已使用普通 `git clone`，请确保已安装 Git LFS 并执行 `git lfs pull`。

### 2. 解压归档文件

将各 tar.gz 解压到正确的目标路径：

```bash
# 解压 JDK 17
tar -xzf jdk-17.tar.gz -C /opt/
# 预期生成目录：/opt/jdk-17/

# 解压 Gradle 8.4
tar -xzf gradle-8.4.tar.gz -C /opt/
# 预期生成目录：/opt/gradle-8.4/

# 解压 Maven 本地仓库
mkdir -p /opt/maven-repo
tar -xzf maven-repo.tar.gz -C /opt/maven-repo/
```

> 解压路径可按需调整，后续环境变量需相应修改。

### 3. 配置环境变量

将以下内容添加到 `~/.bashrc` 或 `~/.zshrc`：

```bash
# JDK 17
export JAVA_HOME=/opt/jdk-17
export PATH=$JAVA_HOME/bin:$PATH

# Gradle 8.4
export GRADLE_HOME=/opt/gradle-8.4
export PATH=$GRADLE_HOME/bin:$PATH

# Android SDK（由 build_env_base 提供）
export ANDROID_HOME=/opt/android-sdk
export PATH=$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH

# Gradle 使用本地 Maven 仓库
export GRADLE_USER_HOME=/opt/maven-repo
```

执行 `source ~/.bashrc` 使配置生效。

### 4. 配置项目 settings.gradle 使用本地 Maven 仓库

在 Android 项目的 `settings.gradle` 或 `settings.gradle.kts` 中添加本地仓库配置：

**Groovy DSL（settings.gradle）：**

```groovy
pluginManagement {
    repositories {
        maven {
            name = "LocalMavenRepo"
            url = uri("/opt/maven-repo")
        }
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven {
            name = "LocalMavenRepo"
            url = uri("/opt/maven-repo")
        }
        google()
        mavenCentral()
    }
}

rootProject.name = "YourProject"
include ':app'
```

**Kotlin DSL（settings.gradle.kts）：**

```kotlin
pluginManagement {
    repositories {
        maven {
            name = "LocalMavenRepo"
            url = uri("/opt/maven-repo")
        }
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        maven {
            name = "LocalMavenRepo"
            url = uri("/opt/maven-repo")
        }
        google()
        mavenCentral()
    }
}

rootProject.name = "YourProject"
include(":app")
```

> 如果项目已有 `pluginManagement` 或 `dependencyResolutionManagement` 块，只需将 `maven` 本地仓库条目添加到现有列表的最前面即可。

### 5. 运行离线编译

```bash
# 确保在 Android 项目根目录下
cd /path/to/your-android-project

# 运行离线组装 Debug APK
./gradlew assembleDebug --offline
```

`--offline` 标志确保 Gradle 不会尝试从网络下载任何依赖，所有依赖均从本地 Maven 仓库获取。

---

## 环境变量配置汇总

| 变量 | 建议值 | 说明 |
|------|--------|------|
| `JAVA_HOME` | `/opt/jdk-17` | JDK 17 安装路径 |
| `GRADLE_HOME` | `/opt/gradle-8.4` | Gradle 8.4 安装路径 |
| `ANDROID_HOME` | `/opt/android-sdk` | Android SDK 路径（由 build_env_base 提供） |
| `GRADLE_USER_HOME` | `/opt/maven-repo` | Gradle 本地缓存/仓库路径 |
| `PATH` | 包含上述各 bin 目录 | 确保命令行工具可用 |

---

## 验证步骤

### 验证 JDK

```bash
java -version
# 预期输出：openjdk version "17.0.x"
```

### 验证 Gradle

```bash
gradle --version
# 预期输出：Gradle 8.4
```

### 验证 Android SDK

```bash
sdkmanager --list
# 应显示已安装的 SDK 包列表
```

### 验证 Maven 本地仓库

```bash
ls /opt/maven-repo/
# 应包含以下典型目录：
# androidx/
# com/
# io/
# net/
# org/
```

### 验证离线编译

```bash
./gradlew assembleDebug --offline --info
# 应能成功完成编译，日志中不应出现网络连接错误
```

---

## 解压后目录结构

```
/opt/
├── jdk-17/                         # JDK 17
│   ├── bin/                        # java, javac 等可执行文件
│   ├── lib/                        # 运行时库
│   ├── conf/                       # 配置文件
│   └── ...
├── gradle-8.4/                     # Gradle 8.4
│   ├── bin/                        # gradle 可执行文件
│   ├── lib/                        # Gradle 运行时库
│   ├── init.d/                     # 初始化脚本
│   └── ...
├── maven-repo/                     # Maven 本地仓库
│   ├── androidx/
│   ├── com/
│   ├── io/
│   ├── net/
│   ├── org/
│   └── ...
└── android-sdk/                    # 由 build_env_base 提供
    ├── cmdline-tools/
    ├── platform-tools/
    ├── platforms/
    ├── build-tools/
    └── ...
```

---

## 相关仓库

| 仓库 | 说明 |
|------|------|
| [build_env_base](https://github.com/your-org/build_env_base) | Android 14 离线构建基础环境（Android SDK + Gradle 缓存） |
| [build_env_add1](https://github.com/your-org/build_env_add1) | **本仓库** — 离线构建补充包（JDK 17 + Gradle 8.4 + Maven 仓库） |
| [build_env_ndk](https://github.com/your-org/build_env_ndk) | NDK 及相关工具链离线包 |

---

## 说明

- 本仓库所有 `*.tar.gz` 文件使用 **Git LFS** 管理，确保大文件的高效存储与传输。
- 本仓库需配合 `build_env_base` 使用，二者共同构成完整的 Android 14 离线构建环境。
- 如有 NDK 编译需求，请额外克隆 `build_env_ndk` 仓库。