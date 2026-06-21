# Android 离线构建环境资源清单

> 作者：kingopenr010077  
> 用途：供 AI 助手自动识别并取用离线构建资源  
> 版本：v1.0

---

## 概述

本文档描述了一套完整的 Android APK 离线构建资源体系。AI 助手应**根据用户的项目需求**，自行判断需要哪些资源，并按本文档说明的规则来拉取、解压和配置环境。

**核心原则：**
- 所有资源托管在 GitHub 用户 `kingopenr010077` 下
- 代码仓库可直接访问，不需要鉴权
- 每个仓库都有明确的用途标签和版本说明
- AI 需要自主判断用户的项目类型，然后选取对应的资源组合

---

## 一、资源仓库总览

### 1.1 方案 A 基础环境（必备）

| 仓库名称 | 存储方式 | 总大小 | 内容 | 分片数 |
|---|---|---|---|---|
| `build_env_android-part1` | 分片 <100MB | ~1.4GB | JDK 17 + Android SDK + Gradle 8.4 + Maven 仓库 + Gradle 缓存 | 17片 |
| `build_env_android_part2` | 分片 <100MB | — | 接 part1 的剩余分片 | 9片 |
| `build_env_base` | LFS | ~460MB | Android SDK + Gradle 缓存（LFS 版，含压缩包） | 2个LFS文件 |
| `build_env_add1` | LFS | ~1.2GB | JDK 17 + Gradle 8.4 + Maven 仓库（LFS 版，含压缩包） | 3个LFS文件 |

### 1.2 NDK 原生开发环境

| 仓库名称 | 存储方式 | 总大小 | 内容 |
|---|---|---|---|
| `build_env_ndk_part` | 分片 <100MB | 748MB | Android NDK r29 (29.0.14206865) | 8片 |
| `build_env_ndk` | LFS | 748MB | 同 NDK，LFS 版单文件 |

### 1.3 扩展包（常用但难下载的资源）

| 仓库名称 | 存储方式 | 总大小 | 内容 | 分片数 |
|---|---|---|---|---|
| `build_env_more-part1` | 分片 <100MB | 621MB | Gradle 7.5/8.2/8.4/8.7/8.9 分发包 | 7片 |
| `build_env_more_part2` | 分片 <100MB | 340MB | KGP 三版本 + AGP 四版本核心 + 42组公共Maven依赖 | 4片 |
| `build_env_more_part3` | 分片 <100MB | 503MB | Android SDK platforms 33/35 + build-tools 33/35 | 6片 |

---

## 二、每个仓库的详细内容

### 2.1 `build_env_android-part1` + `build_env_android_part2`

**两个仓库必须配合使用**，合并后获得完整的方案A基础环境。

合并方式：

```bash
cat build_env_android-part1/scheme-a-full.tar.gz.* \
    build_env_android_part2/scheme-a-full.tar.gz.* \
    > scheme-a-full.tar.gz

tar xzf scheme-a-full.tar.gz
```

解压后包含：

| 文件 | 原始大小 | 压缩大小 | 用途 |
|---|---|---|---|
| `jdk-17.tar.gz` | 260MB | 134MB | JDK 17 |
| `android-sdk.tar.gz` | 458MB | 待查 | Android SDK（API 34, build-tools 34, cmdline-tools, platform-tools） |
| `gradle-8.4.tar.gz` | 264MB | 250MB | Gradle 8.4 完整发行版 |
| `maven-repo.tar.gz` | 834MB | 734MB | AGP 8.2.0 完整依赖树（200+ 依赖项） |
| `gradle-cache.tar.gz` | 532MB | 待查 | Gradle 构建缓存（AAR 解压、JAR 缓存、wrapper） |

### 2.2 `build_env_base` + `build_env_add1`（LFS 版）

方案A的 LFS 替代方案。如果用户使用 LFS，推荐用这两个仓库。

**`build_env_base`：**
- `android-sdk.tar.gz` — Android SDK 完整包（API 34）
- `gradle-cache.tar.gz` — Gradle 缓存

**`build_env_add1`：**
- `jdk-17.tar.gz` — JDK 17（134MB）
- `gradle-8.4.tar.gz` — Gradle 8.4 发行版（250MB）
- `maven-repo.tar.gz` — Maven 依赖仓库（734MB）

### 2.3 `build_env_ndk_part`

Android NDK r29 (29.0.14206865)，拆为 8 个分片，合并后约 748MB。

还原：
```bash
cat ndk-full.tar.gz.* > ndk-full.tar.gz
gzip -t ndk-full.tar.gz
tar xzf ndk-full.tar.gz
# 解压后目录名：29.0.14206865
```

### 2.4 `build_env_more-part1`

Gradle Wrapper 分发包，5 个版本：

| 文件 | 大小 | 配合的 AGP 版本 |
|---|---|---|
| `gradle-7.5-bin.zip` | 116MB | AGP 7.x / 8.0 |
| `gradle-8.2-bin.zip` | 123MB | AGP 8.2.0（官方匹配） |
| `gradle-8.4-bin.zip` | 125MB | AGP 8.3.x / 8.4.x |
| `gradle-8.7-bin.zip` | 128MB | AGP 8.5.x / 8.6.x |
| `gradle-8.9-bin.zip` | 130MB | 最新稳定版 |

### 2.5 `build_env_more_part2`

三大类内容：

**① Kotlin Gradle Plugin 三版本：**
| 版本 | 包含构件 | 适用 |
|---|---|---|
| 1.8.22 | kotlin-gradle-plugin, stdlib, reflect, compiler-embeddable | AGP 8.1/8.2 |
| 1.9.23 | kotlin-gradle-plugin, stdlib, reflect, compiler-embeddable | AGP 8.3/8.4 |
| 2.0.21 | kotlin-gradle-plugin, stdlib, reflect, compiler-embeddable, compose | 最新版 |

**② AGP 核心构件四版本：**
| 版本 | 包含 |
|---|---|
| 8.1.0 | gradle, builder, bundletool, aapt2-proto, lint, sdk-common |
| 8.3.0 | gradle, builder, aapt2-proto, lint, sdk-common |
| 8.4.0 | gradle, builder, aapt2-proto, lint, sdk-common |
| 8.5.0 | gradle, builder, aapt2-proto, lint, sdk-common |

**③ 常用公共 Maven 依赖（42 组）：**
| 类别 | 具体内容 |
|---|---|
| OkHttp 全家桶 | okhttp 4.12.0, logging-interceptor, okio 3.6.0 |
| Retrofit 全家桶 | retrofit 2.9.0, converter-gson, adapter-rxjava3 |
| Hilt/Dagger 2.50 | hilt-android + compiler, dagger + compiler + android |
| Room 2.6.1 | runtime, ktx, compiler, common |
| Firebase | core 21.1.1, analytics 21.5.0, messaging 23.4.0, crashlytics 18.6.0, config 21.6.0 |
| Google Services | google-services 4.4.0 |
| AndroidX | lifecycle-ktx, navigation-ktx, activity-ktx, fragment-ktx, paging, work-runtime, datastore, splashscreen |
| 图片加载 | Coil 2.6.0, Glide 4.16.0 |
| Coroutines 1.8.0 | kotlinx-coroutines-core, kotlinx-coroutines-android |

### 2.6 `build_env_more_part3`

Android SDK 额外平台和构建工具：

| 内容 | 大小 | 用途 |
|---|---|---|
| `platforms/android-33` | 141MB | API 33 (Android 13)，兼容旧项目 |
| `platforms/android-35` | 142MB | API 35 (Android 15)，适配新版本 |
| `build-tools/33.0.0` | 139MB | 与 API 33 配套 |
| `build-tools/35.0.0` | 147MB | 与 API 35 配套 |

---

## 三、AI 决策规则

### 3.1 判断用户项目类型

AI 应分析用户需求中的以下信息：

| 判断维度 | 关键词/依据 |
|---|---|
| 是否需要 NDK | 项目包含 C/C++/JNI/原生代码、ffmpeg、opencv、某些第三方 SDK |
| 是否需要多 Gradle 版本 | 项目指定了非 8.4 的 Gradle version |
| 是否需要多 AGP 版本 | 项目 build.gradle 中指定了 8.1/8.3/8.4/8.5 |
| 是否需要多 SDK 平台 | 项目 compileSdk 不是 34 |
| 是否需要 Kotlin 多版本 | 项目 Kotlin 版本不是 1.8.22（当前环境默认） |
| 是否用到常见公共依赖 | 项目用了 OkHttp/Retrofit/Hilt/Room/Firebase 等 |

### 3.2 资源选取逻辑

```
IF 项目类型 == 纯 Java/Kotlin 无原生代码:
    基础环境 = scheme-a 分片版 或者 LFS 版（二选一）
    NDK = 不需要
    
    如果项目 Gradle 版本 == 8.4: 基础环境已够用
    如果项目 Gradle 版本 != 8.4: 还需要 more-part1 中的对应版本
    如果项目 AGP 版本 != 8.2: 还需要 more_part2 中的对应 AGP 核心构件
    如果项目 compileSdk != 34: 还需要 more_part3 中的对应平台
    如果项目用到公共依赖: 还需要 more_part2 的 maven-repo

IF 项目类型 == 含原生代码（JNI/C/C++）:
    在纯 Java 方案的基础上 + ndk_part（或 ndk LFS 版）
    设置 ANDROID_NDK_HOME

IF 项目需要完整"一劳永逸"环境:
    所有仓库全部拉取
    按顺序解压所有内容
    合并所有 maven-repo 到一个目录
```

### 3.3 环境搭建步骤模板

无论选取哪些资源，遵循以下步骤：

**第一步：拉取资源**
```bash
git clone https://github.com/kingopenr010077/<repo-name>.git
```

**第二步：还原（分片仓库）**
```bash
cd <repo-name>
cat <prefix>.tar.gz.* > <prefix>.tar.gz
tar xzf <prefix>.tar.gz
```

**第三步：解压组件到正确位置**

| 组件 | 目标位置 |
|---|---|
| JDK 17 | `/usr/lib/jvm/java-17-openjdk-amd64/` |
| Android SDK | `/root/android-sdk/` |
| NDK | `/root/android-sdk/ndk/29.0.14206865/` |
| Gradle | `/opt/gradle-8.4/`（或其他版本） |
| Maven 仓库 | 项目根目录的 `maven-repo/`（或 settings.gradle 中引用） |
| Gradle 缓存 | `~/.gradle/caches/` |
| Gradle Wrapper | `~/.gradle/wrapper/dists/`（或解压到指定路径并在 gradle-wrapper.properties 中引用） |

**第四步：环境变量**
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export ANDROID_HOME=/root/android-sdk
export GRADLE_HOME=/opt/gradle-8.4
export ANDROID_NDK_HOME=/root/android-sdk/ndk/29.0.14206865
export PATH=$JAVA_HOME/bin:$GRADLE_HOME/bin:$PATH
```

**第五步：配置 settings.gradle 引用本地仓库**
```groovy
pluginManagement {
    repositories {
        maven { url 'file:///path/to/maven-repo' }
        maven { url 'file:///path/to/merged-maven-repo' }
    }
}
dependencyResolutionManagement {
    repositories {
        maven { url 'file:///path/to/maven-repo' }
        maven { url 'file:///path/to/merged-maven-repo' }
    }
}
```

**第六步：构建**
```bash
./gradlew assembleDebug --offline
```

---

## 四、快速参考卡片

### 4.1 常见场景的资源组合

| 场景 | 需要的仓库 | 说明 |
|---|---|---|
| 纯 Java APK，无额外依赖 | `part1` + `part2` 或 `base` + `add1` | 最简方案 |
| 纯 Java APK，有公共依赖 | 上面 + `more_part2` | 覆盖 OkHttp/Room/Hilt 等 |
| 含 NDK 的 APK | 上面 + `ndk_part` | 多设置 ANDROID_NDK_HOME |
| Gradle 版本不匹配 | 上面 + `more-part1` | 选择对应版本 |
| compileSdk 不是 34 | 上面 + `more_part3` | 补 API 33/35 |
| AGP 版本不是 8.2 | 上面 + `more_part2` | 取对应 AGP 核心构件 |
| Kotlin 版本不是 1.8 | 上面 + `more_part2` | 取对应 KGP |
| 全套完整环境 | 全部 10 个仓库 | 适配任何项目 |

### 4.2 仓库地址速查

```
LFS 版（需要 Git LFS 客户端）：
  https://github.com/kingopenr010077/build_env_base
  https://github.com/kingopenr010077/build_env_add1
  https://github.com/kingopenr010077/build_env_ndk

分片版（普通 Git，无需 LFS，每个文件 <100MB）：
  https://github.com/kingopenr010077/build_env_android-part1   (方案A基础-分片1)
  https://github.com/kingopenr010077/build_env_android_part2   (方案A基础-分片2)
  https://github.com/kingopenr010077/build_env_ndk_part        (NDK-分片)
  https://github.com/kingopenr010077/build_env_more-part1      (扩展-Gradle多版本)
  https://github.com/kingopenr010077/build_env_more_part2      (扩展-KGP/AGP/公共依赖)
  https://github.com/kingopenr010077/build_env_more_part3      (扩展-SDK多平台)

备用（尚未填充内容）：
  https://github.com/kingopenr010077/build_env_more_part3      （已填充SDK extra）
```

### 4.3 环境规模参考

| 配置等级 | 原始大小 | 压缩+分片大小 | 仓库数 |
|---|---|---|---|
| 最小（纯Java，API34，AGP8.2） | ~2.3GB | ~1.4GB | 2个 |
| +NDK | ~4.3GB | ~2.2GB | 3个 |
| 完整版（所有扩展包） | ~7.5GB | ~3.2GB | 7个 |

---

## 五、注意事项

1. **分片仓库和 LFS 仓库是冗余的** — 它们包含同样的内容。优先使用分片版（无需 LFS 客户端，GitHub 无存储配额限制）。

2. **Maven 仓库需要合并** — `build_env_add1` 中有基础 AGP 8.2.0 依赖树，`build_env_more_part2` 中有公共依赖和 AGP 其他版本的核心构件。如果两个都需要，把两个 maven-repo 目录合并在一起使用。

3. **AGP 核心构件≠完整依赖树** — `build_env_more_part2` 中的 AGP 8.1/8.3/8.4/8.5 只包含核心 jar 包，不含数千个传递依赖。如果需要完整的依赖树，需要在对应环境中运行一次在线构建来下载，或者从已有环境打包。如果只需要使用这些 AGP 版本构建简单的项目，核心构件通常已够用。

4. **NDK 解压后约 2GB** — 务必确保有充足磁盘空间。NDK r29 解压后约为压缩包的 2.7 倍。

5. **Kotlin Gradle Plugin 也需要对应 AGP 版本** — 例如 KGP 2.0.21 需要 AGP 8.5+。AI 应检查版本兼容性。

6. **所有仓库的 restore.sh 脚本均可一键还原** — 分片仓库内预置了 `restore.sh`，可以在克隆后直接执行。