# Houdini MPM 雪花爆炸模拟工程分析

## 工程基本信息

- **文件路径**: Z:/houdini/test/other_hip/snowExplosion_MPM.hip
- **Houdini版本**: 21.0.512
- **工程类型**: MPM (Material Point Method) 雪花/颗粒物理模拟

---

## 场景结构概览

```
/obj
├── terrain_mountain (geo)    # 主要几何体容器 - 包含整个模拟系统
├── cam1 (cam)                # 渲染相机1
├── cam2 (cam)                # 渲染相机2
├── cam3 (cam)                # 渲染相机3
├── cam4 (cam)                # 渲染相机4
├── null1 (null)              # 空节点
├── back_1 (geo)              # 背景山体1
└── back_2 (geo)              # 背景山体2
```

---

## 核心技术: MPM Solver

### 什么是MPM?
**Material Point Method (物质点法)** 是一种混合拉格朗日-欧拉方法，特别适用于模拟:
- 雪的堆积和崩塌
- 沙子等颗粒材料
- 泥浆流动
- 大变形固体

MPM结合了粒子系统的灵活性和网格方法的稳定性。

---

## 主节点详解: /obj/terrain_mountain

### 1. HeightField 地形生成系统

| 节点名称 | 类型 | 功能 |
|---------|------|------|
| heightfield1 | heightfield | 基础高度场创建 |
| heightfield_blur1 | heightfield_blur | 高度场模糊处理 |
| heightfield_distort1 | heightfield_distort | 高度场扭曲变形 |
| heightfield_noise1 | heightfield_noise | 添加噪波细节 |

**工作流**: heightfield1 → blur → distort → noise → 最终地形

### 2. MPM 模拟核心组件

#### mpmsolver1 (MPM解算器)
- **类型**: mpmsolver::2.0
- **作用**: 主模拟引擎，执行MPM物理计算
- 内部包含DOP网络进行实际物理解算

#### mpmcontainer1 (MPM容器)
- **类型**: mpmcontainer
- **作用**: 定义模拟空间边界和参数

#### mpmsource1 (MPM粒子源)
- **类型**: mpmsource::2.0
- **作用**: 发射模拟粒子（雪颗粒）

#### mpmcollider1/2/3 (碰撞体)
- **类型**: mpmcollider::2.0
- **作用**: 定义碰撞几何体，粒子与之交互

### 3. 几何处理节点

| 节点 | 功能 |
|-----|------|
| convert1, convert2 | 几何体类型转换 |
| vdbfrompolygons1 | 多边形转VDB体积 |
| convertvolume1 | 体积格式转换 |
| convertvdb1 | VDB转换操作 |
| null_SIM_COLLIDER | 碰撞体输出标记 |

### 4. 输出与缓存系统

| 节点 | 类型 | 用途 |
|-----|------|------|
| SIM | null | 模拟输出标记点 |
| GEO | null | 几何输出标记点 |
| GEO_2 | filecache | 几何缓存2 |
| GEO_3 | filecache | 几何缓存3 |
| GEO_SURFACE | filecache | 表面几何缓存 |
| sim | filecache::2.0 | 模拟数据缓存 |
| ground | filecache::2.0 | 地面缓存 |
| ground_2 | filecache::2.0 | 地面缓存2 |
| geo_surf_cache | filecache::2.0 | 表面几何缓存 |

---

## DOP网络结构 (/obj/terrain_mountain/mpmsolver1/dopnet1)

DOP网络是Houdini动力学模拟的核心：

```
dopnet1
├── mpmobject1      # MPM物体定义
├── mpmsolver       # MPM解算器节点
└── dive_target     # 子网络 - 包含自定义力场
    ├── popwrangle  # VEX代码处理
    └── popforce    # 粒子力场
```

### dive_target 自定义力场
这个子网络包含自定义的物理效果：
- **popwrangle**: 使用VEX代码控制粒子行为（如旋转力、空气阻力）
- **popforce**: 应用额外的力场效果

---

## 背景几何体

### /obj/back_1 和 /obj/back_2
- 用于场景构图的背景山体
- 包含独立的HeightField生成流程
- 主要节点：heightfield、noise、convert等

---

## 工作流程总结

```
1. 地形生成
   heightfield1 → blur → distort → noise → 地形几何体

2. 碰撞体准备
   地形 → convert → vdbfrompolygons → mpmcollider

3. MPM模拟设置
   mpmcontainer (定义模拟空间)
   + mpmsource (发射粒子)
   + mpmcollider (碰撞交互)
   → mpmsolver (物理解算)

4. 后处理与缓存
   模拟结果 → VDB处理 → filecache输出

5. 渲染准备
   多相机设置 (cam1-4) 用于不同角度渲染
```

---

## 关键参数位置

如需调整模拟效果，重点关注：

1. **mpmsolver1** - 模拟精度、时间步长
2. **mpmsource1** - 粒子发射量、初始速度
3. **mpmcollider** - 碰撞摩擦、弹性
4. **mpmcontainer1** - 模拟边界大小
5. **dive_target内的VEX节点** - 自定义力场效果

---

## 学习要点

1. **MPM方法**: 了解物质点法的基本原理
2. **HeightField工作流**: 程序化地形生成
3. **VDB操作**: 体积数据处理
4. **DOP网络**: Houdini动力学系统架构
5. **缓存策略**: 大型模拟的文件管理

---

*笔记生成时间: 2026-01-04*
*分析工具: Claude Code + Houdini MCP*
