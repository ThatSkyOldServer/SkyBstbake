# Sky-Bstbake

> A tool for unpacking Sky: Children of the Light map baked data (`.meshes`) and exporting OBJ models

## Supported Versions (Verified)
- LVL04 - LVL0D
- sky-light-await (0.1.8) - sky-children-of-the-light (0.33.5)

## Authors & Module Credits

| Item | Description |
|------|-------------|
| **Authors** | checion (雨人) & Heriel (落秋) |
| **External Dependency** | [meshoptimizer](https://github.com/zeux/meshoptimizer) git submodule — built by CMake for vertex/index buffer decompression |
| **Runtime** | Python 3.6+ |

---

## Feature Overview

`Sky-Bstbake.py` parses and unpacks the game engine's **LVL0 format map baked data files** (`.meshes`), splitting the geometry data into individual segments and exporting them as standard OBJ model files for viewing and editing in 3D software such as Blender.

### Core Features

| Feature | Description |
|---------|-------------|
| **LVL0 File Parsing** | Reads file header, TOC (Table of Contents), and locates each data segment |
| **LZ4 Decompression** | Decompresses the LOD0 segment via LZ4 block decompression to restore raw geometry data |
| **Meshopt Decoding** | Decodes meshopt-compressed vertex/index buffers through a C dynamic library |
| **Multi-Segment Splitting** | Automatically splits Mesh / Terrain / Cloud / Skirt / Occluder / METR / GEO0 |
| **OBJ Model Export** | Exports Terrain, Skirt, Occluder, and Cloud Proxy as `.obj`  |

### Supported Data Segments

| Segment | Description |
|---------|-------------|
| `Mesh.bin` | Map mesh baked data (names, GUIDs, submesh indices) |
| `Terrain.bin` | Terrain vertices + indices + octree + grid culling + hardware tessellation |
| `GEO0.bin` | v57+ terrain data (meshopt-compressed global vertex buffer + byte triplet indices) |
| `Cloud.bin` | Volumetric cloud sparse block data (distance field / ambient light / proxy mesh) |
| `Skirt.bin` | Skirt geometry (terrain edge transition mesh with UV and tangent) |
| `Occluder.bin` | Occlusion culling volume mesh (CPU-side frustum optimization) |
| `METR.bin` | Segment index and memory allocation metadata |

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
├── GEO0.bin                # v57+ terrain data
├── input.obj               # Exported 3D model
```

### 4. OBJ Export Contents

The exported OBJ file contains the following objects:

- **Terrain_N** — Terrain chunks (v57+ grouped by A/B patch arrays)
- **CloudProxy_N** — Volumetric cloud proxy mesh
- **Skirt_N** — Skirt transition mesh
- **Occluder_N** — Occlusion culling volume
- **CloudProxy** — Legacy standalone cloud proxy

---

### Version Differences

| Version | Key Changes |
|---------|-------------|
| LVL04+ | Cloud block adds a 1-byte flag |
| LVL05+ | Adds METR segment, Cloud mask changes to 2 bytes/pixel, Skirt uses meshopt compression |
| LVL07+ | Adds GEO0 segment (meshopt-compressed terrain), Terrain data migrates to GEO0 |
| LVL0B+ | Cloud removes bitmask, switches to 4 compressed chunks |

---

## License & Acknowledgments

- Vertex/index decompression relies on [meshoptimizer](https://github.com/zeux/meshoptimizer) (MIT License)
- LZ4 decompression uses [python-lz4](https://github.com/python-lz4/python-lz4)




# Sky-Bstbake

> 光遇游戏地图烘焙数据（`.meshes`）解包与 OBJ 模型导出工具


## 支持的版本范围（经过验证）
- LVL04 - LVL0D
- sky-light-await(0.1.8) - sky-children-of-the-light(0.33.5)


## 作者与模块声明

| 项目 | 说明 |
|------|------|
| **作者** | 雨人 (checion) 与 落秋 (Heriel) |
| **外部依赖模块** | [meshoptimizer](https://github.com/zeux/meshoptimizer) Git 子模块 — 由 CMake 编译用于顶点/索引缓冲解压缩 |
| **运行环境** | Python 3.6+ |

---

## 功能概述

`Sky-Bstbake.py` 用于解析和解包游戏引擎的 **LVL0 格式地图烘焙数据文件**（`.meshes`），将其中的几何数据拆分为独立段并导出为标准 OBJ 模型文件，便于在 Blender 等 3D 软件中查看和编辑。

### 核心功能

| 功能 | 描述 |
|------|------|
| **LVL0 文件解析** | 读取文件头、TOC（目录表），定位各数据段 |
| **LZ4 解压** | 对 LOD0 段进行 LZ4 块解压，还原原始几何数据 |
| **Meshopt 解码** | 通过 C 动态库解码 meshopt 压缩的顶点/索引缓冲 |
| **多段数据拆分** | 自动拆分 Mesh / Terrain / Cloud / Skirt / Occluder / METR / GEO0 |
| **OBJ 模型导出** | 将 Terrain、Skirt、Occluder、Cloud Proxy 导出为 `.obj` |

### 支持的数据段

| 段名 | 说明 |
|------|------|
| `Mesh.bin` | 地图网格烘焙数据（名称、GUID、子网格索引） |
| `Terrain.bin` | 地形顶点 + 索引 + 八叉树 + 网格剔除 + 硬件细分曲面 |
| `GEO0.bin` | v57+ 地形数据（meshopt 压缩的全局顶点缓冲 + 字节三元组索引） |
| `Cloud.bin` | 体积云稀疏块数据（距离场 / 环境光 / 代理网格） |
| `Skirt.bin` | 裙边几何（地形边缘过渡网格，含 UV 和切线） |
| `Occluder.bin` | 遮挡剔除体积网格（CPU 端视锥体优化） |
| `METR.bin` | 段索引与内存分配元数据 |

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
├── GEO0.bin                # v57+ 地形数据
├── input.obj               # 导出的 3D 模型
```

### 4. OBJ 导出内容

导出的 OBJ 文件包含以下对象：

- **Terrain_N** — 地形块（v57+ 按 A/B Patch 阵列分组）
- **CloudProxy_N** — 体积云代理网格
- **Skirt_N** — 裙边过渡网格
- **Occluder_N** — 遮挡剔除体积
- **CloudProxy** — 旧版本独立云代理

---

### 版本差异

| 版本 | 主要变化 |
|------|----------|
| LVL04+ | Cloud块新增一字节flag |
| LVL05+ | 新增 METR 段，Cloud mask 改为 2字节/像素，Skirt 使用 meshopt 压缩 |
| LVL07+ | 新增 GEO0 段（meshopt 压缩地形），Terrain 数据迁移至 GEO0 |
| LVL0B+ | Cloud 移除 bitmask，改为 4 个压缩 chunk |

---

## 许可与致谢

- 顶点/索引解压依赖 [meshoptimizer](https://github.com/zeux/meshoptimizer) (MIT License)
- LZ4 解压使用 [python-lz4](https://github.com/python-lz4/python-lz4)
