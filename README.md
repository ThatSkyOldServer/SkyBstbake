# Sky-Bstbake

> A tool for unpacking Sky: Children of the Light map baked data (`.meshes`) and exporting OBJ models

## Supported Versions
- LVL04 - LVL0= (file version 52 - 61)
- sky-light-await (0.1.8) - sky-children-of-the-light (0.35.0)

## Authors & Module Credits

| Item | Description |
|------|-------------|
| **Authors** | checion (雨人) & Heriel (落秋) |
| **External Dependency** | [meshoptimizer](https://github.com/zeux/meshoptimizer) git submodule — built by CMake for vertex/index buffer decompression |
| **Runtime** | Python 3.6+ |

---

## Feature Overview

`Sky-Bstbake.py` parses and unpacks the game engine's **LVL0 format map baked data files** (`.meshes`), splitting the geometry data into individual segments and exporting them as standard OBJ model files for viewing and editing in 3D software such as Blender.

This repository only demonstrates the `.meshes` structure in a simple, readable way so the format can be understood. It does not provide anything beyond that.

### Core Features

| Feature | Description |
|---------|-------------|
| **LVL0 File Parsing** | Reads file header, TOC (Table of Contents), and locates each data segment |
| **LZ4 Decompression** | Decompresses the LOD0 segment via LZ4 block decompression to restore raw geometry data |
| **Meshopt Decoding** | Decodes meshopt-compressed vertex/index buffers through a C dynamic library |
| **Multi-Segment Splitting** | Automatically splits GEO0 / WayfinderRegion / Mesh / Terrain / Cloud / Skirt / Occluder / METR |
| **OBJ Model Export** | Exports Terrain, Skirt, Occluder, and Cloud Proxy as `.obj`  |

### Supported Data Segments

| Segment | Description |
|---------|-------------|
| `Mesh.bin` | Map mesh baked data (names, GUIDs, submesh indices) |
| `Terrain.bin` | Terrain vertices + indices + octree + grid culling + hardware tessellation |
| `GEO0.bin` | v56+ terrain data; v56 uses meshopt index buffers, v57+ uses a meshopt global vertex buffer plus per-patch byte-triplet indices |
| `WayfinderRegion.bin` | v61+ baked navigation grid, split out of the GEO0 tail (`u32 regionCount`, then per region `u32 cellCount` and 20-byte cells) |
| `Cloud.bin` | Volumetric cloud sparse block data (distance field / ambient light / proxy mesh) |
| `Skirt.bin` | Skirt geometry; v57+ rows are 36 bytes, older meshopt rows are 40 bytes with a `a_userData1` tail |
| `Occluder.bin` | Occlusion culling volume mesh (CPU-side frustum optimization) |
| `METR.bin` | 64-byte level-bake metrics block (version, size, bake time, file size, per-chunk geometry counters) |

---

## Usage

### 1. Environment Setup

```bash
# Install Python dependency
pip install lz4

# Clone the repository together with the meshoptimizer submodule
git clone --recurse-submodules https://github.com/ThatSkyOldServer/SkyBstbake
cd SkyBstbake

# For an existing checkout, initialize/update the submodule instead
git submodule update --init --recursive

# Build the platform-native shared library (CMake 3.15+)
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

The build places the library in `_meshopt/lib/`. The filename is selected
automatically by the Python loader: `meshoptimizer.dll` on Windows,
`libmeshoptimizer.dylib` on macOS, and `libmeshoptimizer.so` on Linux and
other supported Unix-like platforms.

### 2. Basic Usage

```bash
# Unpack and split only (generates individual .bin segment files)
python Sky-Bstbake.py --unpack <input.meshes>

# Unpack + export OBJ model
python Sky-Bstbake.py --unpack <input.meshes> --export-obj
```

### 3. Output Structure

```
input/                      # Directory named after the input file
├── Mesh.bin                # Mesh baked data
├── Terrain.bin             # Terrain data
├── Cloud.bin               # Volumetric cloud data
├── Skirt.bin               # Skirt data
├── Occluder.bin            # Occlusion culling data
├── METR.bin                # Metadata (v55+)
├── GEO0.bin                # v56+ terrain data
├── WayfinderRegion.bin     # v61+ navigation grid (when present)
├── input.obj               # Exported 3D model
├── input_navmesh.obj       # Navigation grid only (v61+, when present)
```

### 4. OBJ Export Contents

The exported OBJ file contains the following objects:

- **Terrain_N** — Terrain chunks (grouped by A/B patch arrays)
- **CloudProxy_N** — Volumetric cloud proxy mesh
- **Skirt_N** — Skirt transition mesh
- **Occluder_N** — Occlusion culling volume
- **CloudProxy** — Legacy standalone cloud proxy
- **NavMesh_N** / **NavMesh_N_Border** — Navigation grid cells and their border cells (v61+)

---

### Version Differences

| Version | Key Changes |
|---------|-------------|
| LVL06+ (v54+) | Cloud proxy gains one extra byte |
| LVL07+ (v55+) | Adds METR segment, Cloud mask becomes 2 bytes/pixel, Skirt uses meshopt compression |
| LVL08+ (v56+) | Adds GEO0 segment (meshopt-compressed terrain), Terrain data migrates to GEO0 |
| LVL09+ (v57+) | GEO0 switches to a meshopt global vertex buffer plus per-patch byte-triplet indices; Skirt rows shrink to 36 bytes |
| LVL0<+ (v60+) | Cloud drops the bitmask and switches to 4 compressed chunks |
| LVL0=+ (v61+) | GEO0 gains a WayfinderRegion navigation grid after Array C |

---

## License & Acknowledgments

- Vertex/index decompression relies on [meshoptimizer](https://github.com/zeux/meshoptimizer) (MIT License)
- LZ4 decompression uses [python-lz4](https://github.com/python-lz4/python-lz4)




# Sky-Bstbake

> 光遇游戏地图烘焙数据（`.meshes`）解包与 OBJ 模型导出工具


## 支持的版本范围
- LVL04 - LVL0=（文件版本 52 - 61）
- sky-light-await(0.1.8) - sky-children-of-the-light(0.35.0)


## 作者与模块声明

| 项目 | 说明 |
|------|------|
| **作者** | 雨人 (checion) 与 落秋 (Heriel) |
| **外部依赖模块** | [meshoptimizer](https://github.com/zeux/meshoptimizer) Git 子模块 — 由 CMake 编译用于顶点/索引缓冲解压缩 |
| **运行环境** | Python 3.6+ |

---

## 功能概述

`Sky-Bstbake.py` 用于解析和解包游戏引擎的 **LVL0 格式地图烘焙数据文件**（`.meshes`），将其中的几何数据拆分为独立段并导出为标准 OBJ 模型文件，便于在 Blender 等 3D 软件中查看和编辑。

本仓库仅以简单直白的方式展示 `.meshes` 的结构以便理解，不提供更多内容。

### 核心功能

| 功能 | 描述 |
|------|------|
| **LVL0 文件解析** | 读取文件头、TOC（目录表），定位各数据段 |
| **LZ4 解压** | 对 LOD0 段进行 LZ4 块解压，还原原始几何数据 |
| **Meshopt 解码** | 通过 C 动态库解码 meshopt 压缩的顶点/索引缓冲 |
| **多段数据拆分** | 自动拆分 GEO0 / WayfinderRegion / Mesh / Terrain / Cloud / Skirt / Occluder / METR |
| **OBJ 模型导出** | 将 Terrain、Skirt、Occluder、Cloud Proxy 导出为 `.obj` |

### 支持的数据段

| 段名 | 说明 |
|------|------|
| `Mesh.bin` | 地图网格烘焙数据（名称、GUID、子网格索引） |
| `Terrain.bin` | 地形顶点 + 索引 + 八叉树 + 网格剔除 + 硬件细分曲面 |
| `GEO0.bin` | v56+ 地形数据；v56 使用 meshopt 索引缓冲，v57+ 使用 meshopt 全局顶点缓冲 + 每 patch 的字节三元组索引 |
| `WayfinderRegion.bin` | v61+ 烘焙导航网格，从 GEO0 尾部单独拆出（`u32 regionCount`，每个区域 `u32 cellCount` 与 20 字节 cell） |
| `Cloud.bin` | 体积云稀疏块数据（距离场 / 环境光 / 代理网格） |
| `Skirt.bin` | 裙边几何；v57+ 顶点为 36 字节，更早的 meshopt 版本为 40 字节并带 `a_userData1` 尾部 |
| `Occluder.bin` | 遮挡剔除体积网格（CPU 端视锥体优化） |
| `METR.bin` | 64 字节关卡烘焙统计块（版本、大小、烘焙耗时、文件大小、各段几何计数） |

---

## 使用方法

### 1. 环境准备

```bash
# 安装 Python 依赖
pip install lz4

# 克隆仓库时同时获取 meshoptimizer 子模块
git clone --recurse-submodules https://github.com/ThatSkyOldServer/SkyBstbake
cd SkyBstbake

# 已有工作区则初始化/更新子模块
git submodule update --init --recursive

# 使用 CMake 编译当前平台的动态库（CMake 3.15+）
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

动态库会生成到 `_meshopt/lib/`。Python 会按平台自动选择文件名：Windows
使用 `meshoptimizer.dll`，macOS 使用 `libmeshoptimizer.dylib`，Linux 及其他支持的
类 Unix 平台使用 `libmeshoptimizer.so`。

### 2. 基本用法

```bash
# 仅解包拆分（生成各 .bin 段文件）
python Sky-Bstbake.py --unpack <input.meshes>

# 解包 + 导出 OBJ 模型
python Sky-Bstbake.py --unpack <input.meshes> --export-obj
```

### 3. 输出结构

```
input/                      # 与输入文件同名的目录
├── Mesh.bin                # 网格烘焙数据
├── Terrain.bin             # 地形数据
├── Cloud.bin               # 体积云数据
├── Skirt.bin               # 裙边数据
├── Occluder.bin            # 遮挡剔除数据
├── METR.bin                # 元数据（v55+）
├── GEO0.bin                # v56+ 地形数据
├── WayfinderRegion.bin     # v61+ 导航网格（存在时）
├── input.obj               # 导出的 3D 模型
├── input_navmesh.obj       # 仅导航网格（v61+，存在时）
```

### 4. OBJ 导出内容

导出的 OBJ 文件包含以下对象：

- **Terrain_N** — 地形块（按 A/B Patch 阵列分组）
- **CloudProxy_N** — 体积云代理网格
- **Skirt_N** — 裙边过渡网格
- **Occluder_N** — 遮挡剔除体积
- **CloudProxy** — 旧版本独立云代理
- **NavMesh_N** / **NavMesh_N_Border** — 导航网格格与其边界格（v61+）

---

### 版本差异

| 版本 | 主要变化 |
|------|----------|
| LVL06+ (v54+) | Cloud 代理块新增一个字节 |
| LVL07+ (v55+) | 新增 METR 段，Cloud mask 改为 2 字节/像素，Skirt 使用 meshopt 压缩 |
| LVL08+ (v56+) | 新增 GEO0 段（meshopt 压缩地形），Terrain 数据迁移至 GEO0 |
| LVL09+ (v57+) | GEO0 改用 meshopt 全局顶点缓冲 + 每 patch 字节三元组索引；Skirt 顶点缩为 36 字节 |
| LVL0<+ (v60+) | Cloud 移除 bitmask，改为 4 个压缩 chunk |
| LVL0=+ (v61+) | GEO0 在 Array C 之后新增 WayfinderRegion 导航网格 |

---

## 许可与致谢

- 顶点/索引解压依赖 [meshoptimizer](https://github.com/zeux/meshoptimizer) (MIT License)
- LZ4 解压使用 [python-lz4](https://github.com/python-lz4/python-lz4)
