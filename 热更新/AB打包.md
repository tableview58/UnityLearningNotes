# **AssetBundle 打包工具**

------

一个 **Unity 编辑器扩展**，有界面的一键工具，并且支持 **命令行** 在 CI/CD 中自动执行。构建后会自动生成 **版本信息**、**文件清单（含 MD5）** 和 **依赖分析** 等热更需要的元数据。



### 1.在项目中的作用

- **热更/资源发布**：产物带 version.txt、filelist.json，方便差量更新对比与 CDN 发布。
- **流水线构建**：用命令行接口嵌入 Jenkins / GitHub Actions，自动化夜构或提测构建。
- **依赖排查**：dependencies.json 用于分析资源依赖、拆包与去重。

------

## **2. 核心功能点**

- **多平台打包**：为 **Windows/Android/iOS** 输出各自目录。
- **压缩策略可选**：**LZ4 / LZMA / Uncompressed**（UI 与 CLI 映射一致）。
- **版本与元数据**：输出 version.txt、filelist.json（bundle 大小、MD5、直依赖）。
- **依赖分析（资源级）**：dependencies.json 记录 asset → 直接依赖。
- **清理输出**：一键清空当前平台输出目录。
- **命令行构建**：-buildPlatform/-outputPath/-compression 参数化执行。
- **配置持久化**：ScriptableObject 保存最近一次平台/路径/压缩/版本号。

------

## **3. 使用到的技术 / Unity API**

- BuildPipeline.BuildAssetBundles(outDir, options, target)（**核心打包**）
  - **解释**：根据资源 Importer 上的 **AssetBundle Name** 把资源分组、构建为若干包；返回 AssetBundleManifest。
  - **原理**：构建管线收集所有打包条目→解析依赖→确定每个包的资源与共享资源→生成二进制包与清单。
- AssetBundleManifest（**包清单**）
  - **解释**：提供 GetAllAssetBundles()、GetDirectDependencies(bundle) 等查询接口。
  - **原理**：Manifest 是打包阶段产出的依赖图索引，供编辑器或运行时决定加载顺序、预加载 shared 资源。
- AssetDatabase.GetAllAssetBundleNames() / GetAssetPathsFromAssetBundle()（**查询标签**）
  - **解释**：获取全部 AB 名称/某包中包含的资源路径。
  - **原理**：读取导入设置与 GUID-Path 映射表，返回编辑器数据库中记录的分组信息。
- AssetDatabase.GetDependencies(assetPath, recursive:false)（**资源依赖**）
  - **解释**：获取一个资源的**直接**依赖（不递归）。
  - **原理**：基于导入时建立的引用关系图（GUID 级别）向外一层展开。
- EditorWindow / EditorGUILayout / MenuItem（**自定义窗口**）
  - **解释**：创建“AssetBundle Builder”窗口与菜单入口，绘制 UI。
  - **原理**：IMGUI 事件驱动，OnGUI() 每帧根据状态重绘控件。
- EditorUtility.DisplayProgressBar/ClearProgressBar（**进度提示**）
  - **解释**：构建/分析时显示阶段进度与提示文本。
  - **原理**：编辑器线程 UI API；需要在 finally 清理。
- ScriptableObject + AssetDatabase.CreateAsset/SaveAssets（**配置持久化**）
  - **解释**：把最近一次构建参数存到 *.asset，UI 与 CLI 共用。
  - **原理**：把序列化对象存成工程资源（非场景），由 AssetDatabase 管理。
- Environment.GetCommandLineArgs()（**读取 CLI 参数**）
  - **解释**：解析 -buildPlatform 等键值参数。
  - **原理**：进程启动参数；解析后映射到枚举与路径。
- MD5（System.Security.Cryptography，**产物指纹**）
  - **解释**：计算 bundle 文件的 MD5 值写入清单。
  - **原理**：对文件字节流做哈希；生产可考虑升级为 SHA256。

------

## **4. 实现流程**

### **A. 流程图（Mermaid）**

```
flowchart TD
  UI[打开窗口/读取配置] --> Set[选择平台/路径/压缩/版本]
  Set --> Build{点击 Build?}
  Build -- 否 --> Clean{点击 Clean?}
  Clean -- 是 --> Del[删除平台输出目录] --> UI
  Clean -- 否 --> Analyze{点击 Analyze?}
  Analyze -- 是 --> Dep[扫描 AB 名称/资源→直依赖] --> OutDep[写 dependencies.json] --> UI
  Analyze -- 否 --> UI
  Build -- 是 --> Prep[创建 <output>/<Platform>]
  Prep --> BP[BuildPipeline.BuildAssetBundles]
  BP --> Meta[写 version.txt / filelist.json]
  Meta --> Stat[统计大小/耗时/日志]
  Stat --> Bump[版本号 +1; 保存配置]
  Bump --> Done[完成]
```

### **B. 伪代码（核心路径）**

```
// UI Build
LoadConfig();
outDir = Path.Combine(outputRoot, PlatformName(target));
EnsureDirectory(outDir);
manifest = BuildPipeline.BuildAssetBundles(outDir, compression, target);
if (manifest == null) throw;
WriteVersion(outDir, versionNumber);
WriteFileList(outDir, manifest);
versionNumber++;
SaveConfig();

// CLI Build
ParseArgs();
SyncToConfig();
EnsureDirectory(outDir);
manifest = BuildPipeline.BuildAssetBundles(outDir, compression, target);
WriteMetadata(outDir, manifest, config.versionNumber);
config.versionNumber++;
config.Save();
SaveAssets(); Exit(0);
```

### **C. 初学者视角（像教程）**

1. 给资源设置 **AssetBundle Name**。
2. 打开窗口，选 **平台**、**输出路径**、**压缩方式**、**版本号**。
3. 点 **Build**，等进度条结束；看控制台和日志区域。
4. 在输出目录里找到 version.txt、filelist.json；这些给“下载器/热更”用。
5. 想清空旧产物 → 点 **Clean**；想看资源互相引用 → 点 **Analyze**。

### **D. 高级开发者视角（执行机制）**

- 构建入口统一走 BuildPipeline.BuildAssetBundles，以 **Importer 上的 AB Name** 做分组依据。
- 生成的 AssetBundleManifest 是包级 DAG；filelist.json 记录直依赖以方便运行时拓扑排序。
- 版本号来自同一个 ScriptableObject，UI 与 CLI 打完**都会自增并持久化**，实现“单一真相源”。
- dependencies.json 在编辑期用 AssetDatabase.GetDependencies 生成**资源级**关系图，用于审计与重构，而非运行时加载必须项。



------

## **5. 架构与设计模式分析**

### **设计思路**

- **三段式结构**：Config（状态持久化） / EditorWindow（交互与流程编排） / CLI（无头构建）。
- **关注点分离**：UI 不处理 CLI 解析；CLI 不依赖窗口；两者共享配置与构建逻辑的一致性。

### **使用的模式**

- **单例（变体）**：AssetBundleBuilderConfig.Instance 通过固定路径 *.asset 懒加载，表现为“Asset 单例”。
- **门面（Facade）**：EditorWindow 充当门面统一调用打包/生成清单/统计等步骤。

### **为什么这么设计 & 替代方案**

- **为什么**：简单直接、接入成本低；现有团队可快速落地。
- **替代**：
  - 引入 **服务层**（纯 C# BuilderService）承载构建与元数据生成，UI/CLI 仅作壳（便于单元测试与扩展）。
  - 使用 **Scriptable Build Pipeline (SBP)** 与 **Addressables**，可插拔任务、增量构建更强。
  - 配置改为 **Profile 集**（多环境），通过 -profile 切换。

------

## **6. 性能与优化分析**

### **影响维度**

- **速度**：资源导入（首次/切平台）、依赖分析、压缩算法（LZMA 慢/LZ4 快）。
- **内存**：大工程打包时峰值内存可能高，建议 64-bit 编辑器与充足虚拟内存。
- **磁盘**：输出体积与 Library/ 缓存；CI 机器注意磁盘清理策略。

### **可能的瓶颈**

- 频繁切换 BuildTarget 触发 reimport。
- LZMA 压缩阶段 CPU 吃紧、耗时长。
- 超大工程的 dependencies.json 写入与序列化。

### **优化建议**

- **压缩策略**：分发用 LZMA，首启后转换/缓存 LZ4；或直接 LZ4 以提升加载体验。
- **增量打包**：对比上次 filelist.json，仅重打改动包；或迁移 **SBP** 启用缓存。
- **缓存与并行**：配置 Unity Cache Server；使用多核 CI 节点。
- **依赖输出降噪**：改为 **bundle 级**报告或增加体量阈值/采样。
- **AppendHash**：启用 AppendHashToAssetBundleName（需改客户端加载）以提升 CDN 命中与回滚能力。

------

## **7. 优缺点总结**

### **面向新手**

- **优点**：按钮少、步骤清、结果齐（版本/清单/依赖）。
- **缺点**：必须先给资源设置 AB Name；压缩选项含义需要理解。

### **技术视角**

- **优点**：UI/CLI 同源配置、版本自增；日志/异常/目录保障完善；可无缝接入 CI。
- **缺点**：基于经典 BuildPipeline，缺乏增量；依赖分析为资源级，体量可能大；未启用哈希命名。

------

## **8. 改进方向与扩展思路**

- **批量/矩阵打包**：一次性对多个平台与压缩维度做遍历构建；产出对照表。
- **配置管理**：用 ScriptableObject 定义 **Profile**（dev/staging/prod），CLI 传 -profile 选择；支持 JSON 导入导出。
- **SBP/Addressables 迁移**：以任务图方式重构构建流程，开启增量与缓存，兼容更复杂依赖与可寻址策略。
- **上传与发布**：构建后自动上传到对象存储（S3/OSS/GCS）与生成索引页；写入 sha256 与 crc。
- **校验与回滚**：服务端比对 filelist.json 差量；客户端基于版本与哈希回滚。

------

## **9. 原理深度剖析**

### **AssetBundle 的存储结构（高层）**

- **Header**：格式版本与平台标识。
- **BlockInfo**：压缩/块划分信息（LZ4/LZMA/无压缩）。
- **Directory / Table**：内部对象与资源索引表。
- **Data**：实际资源数据块。

### **Unity 如何管理依赖与加载**

- **编辑期**：导入时记录资源 GUID 与依赖；给资源标记 AB Name 后，构建器基于全局依赖图做“包内聚与共享抽取”。
- **运行时**：根据 Manifest 决定加载顺序；若 A 依赖 B，需先加载 B 或依赖被自动拉起（取决于加载策略）。

### **打包管线的生命周期（经典 BuildPipeline）**

1. **收集条目**：扫描所有设置了 AB Name 的资源。
2. **解析依赖**：构建全局依赖图，划分包与共享资源。
3. **写包**：根据压缩策略输出二进制包与 *.manifest 文件。
4. **写全局 Manifest**：生成 AssetBundleManifest 索引表。
5. **清理/缓存**：写入 Library 缓存（受版本/平台/导入设置影响）。

------



### **附：压缩方式对比（一图速览）**



| **压缩**     | **选项映射**            | **体积** | **解压/加载**  | **适用场景**         |
| ------------ | ----------------------- | -------- | -------------- | -------------------- |
| LZ4          | ChunkBasedCompression   | 中       | 快             | 移动端/频繁加载/流式 |
| LZMA         | None(默认 LZMA)         | 小       | 慢             | 分发体积敏感/首发包  |
| Uncompressed | UncompressedAssetBundle | 大       | 很快（无解压） | 调试/带宽与存储充裕  |