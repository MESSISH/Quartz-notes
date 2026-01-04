# Houdini Particle Project Notes

**工程文件**: `Z:/houdini/work/particle_v007.hip`
**Houdini版本**: 21.0.512
**整理日期**: 2026-01-04

---

## 1. 工程概述

这是一个粒子特效工程，用于生成场景中物体（狗、球、秋千、茶几椅子、地面等）的粒子动画效果，并按区域分层输出Alembic文件。

---

## 2. 场景结构 (/obj)

| 节点名 | 类型 | 用途 |
|--------|------|------|
| `scence` | geo | 场景几何体导入与处理 |
| `particle` | geo | **核心工作区** - 粒子生成与动画 |
| `dog_anim` | geo | 狗动画导入与处理 |
| `alembicarchive1` | alembicarchive | 相机ABC源 |
| `null2` | null | 相机archive父节点 |
| `cam_Frame1` | cam | 第1帧相机位置 |
| `cam_Frame200` | cam | 第200帧相机位置 |
| `topnet1` | topnet | 渲染输出TOP网络 |

---

## 3. 模块详解

### 3.1 场景模块 (scence)

```
file1 → transform1 → attribdelete1 → filecache_geo → timeshift1 → OUT_ALL_Geo
                                                   ↓
                                               convert1
```

- 从文件导入场景几何体
- 清理不需要的属性后缓存

### 3.2 狗动画模块 (dog_anim)

```
alembic1 → transform1 → unpack1 → clean2 → filecache_dog_and_ball → Out_dog_and_ball
                                    ↓
                                  dog (grouppromote)
                                    ↓
                               dog_ball (groupcreate)

file1 → unpack2 → convert2 → clean3 → blast1 → transform2 ─┐
                                                           ↓
                                                        merge1
```

- 导入狗和球的Alembic动画
- 创建dog/dog_ball分组

### 3.3 粒子核心模块 (particle) - 321个节点（整理后）

#### 3.3.1 数据输入
- `get_ALL_Geo` - 获取场景几何体
- `get_aim_cam` / `get_Frame1_cam` / `get_Frame200_cam` - 获取相机
- `get_dog_and_ball` / `get_dog_rest` - 获取狗动画数据

#### 3.3.2 区域分组
| 分组节点 | 区域 |
|----------|------|
| `_tea_chair` | 茶几椅子区域 |
| `_swing` | 秋千区域 |
| `_ground` | 地面区域 |
| `_tea_t` / `_tea_table` | 茶几/茶桌 |
| `_dog` / `_ball` | 狗/球 |
| `_scene` | 场景总体 |

#### 3.3.3 粒子生成 (Scatter)
- `scatter1` - 主场景散点
- `scatter4` - 狗身散点
- `scatter5` - 前景散点
- `scatter6/7` - 茶几区域散点
- `scatter8` - 地面散点
- `scatter9` - 其他区域散点

#### 3.3.4 动画处理 (Solver)
| Solver | 用途 |
|--------|------|
| `solver1` | 主粒子求解器 |
| `solver2` | 辅助求解器 |
| `solver2_2d` | 2D投影求解 |
| `solver3` | 线条生成求解 |
| `solver3_3d` | 3D线条求解 |

#### 3.3.5 缓存节点 (FileCache)
| 缓存节点 | 内容 |
|----------|------|
| `filecache_geo_swing` | 秋千几何体缓存 |
| `filecache_geo_tea_chair` | 茶几椅子缓存 |
| `filecache_cam_geo` | 相机几何体缓存 |
| `filecache_geo_visibl_del` | 可见性处理缓存 |
| `filecache_scatter` | 散点缓存 |
| `filecache_pt_attrib_dist` | 点属性距离缓存 |
| `filecache_anim_pt` | 动画点缓存 |
| `filecache_dog_scatter` | 狗散点缓存 |
| `filecache_dog_scatter1` | 狗散点缓存(备份) |
| `filecache_dog_full_pt` | 狗完整点缓存 |

#### 3.3.6 输出 (ROP Alembic)
| ROP节点 | 输出内容 |
|---------|----------|
| `ball_particle` | 球粒子 |
| `dog_particle` | 狗粒子 |
| `foreground_particle` | 前景粒子 |
| `vista_particle` | 远景粒子 |
| `ground_particle` | 地面粒子 |
| `swing_particle` | 秋千粒子 |
| `tea_chair_particle` | 茶几椅子粒子 |
| `scene_particle` | 场景粒子 |
| `line_geo` | 线条几何体 |

#### 3.3.7 批量输出
- `topnet_batch_output` - TOP网络批量渲染所有粒子输出

---

## 4. 工作流程

```
1. 数据导入
   ├── scence: 场景几何体
   ├── dog_anim: 狗动画
   └── alembicarchive1: 相机

2. 区域划分与分组
   └── particle: 各区域groupcreate

3. 粒子散布
   └── scatter节点在各区域生成点

4. 动画求解
   └── solver系列处理粒子运动

5. 缓存优化
   └── filecache节点保存中间结果

6. 分层输出
   └── rop_alembic按区域导出ABC
```

---

## 5. 整理记录

### /obj层级删除节点（2个）
| 节点 | 类型 | 原因 |
|------|------|------|
| `/obj/______` | alembicarchive | 废弃相机资源，命名异常 |
| `/obj/null1` | null | 仅连接到废弃节点 |

### /obj/particle内部删除节点（17个）
| 节点 | 类型 | 原因 |
|------|------|------|
| `sphere7-12` | sphere | 完全孤立，无连接 |
| `sphere13-17` | sphere | 完全孤立，无连接 |
| `split4` | split | 完全孤立 |
| `timeshift14` | timeshift | 完全孤立 |
| `attribwrangle33` | attribwrangle | 完全孤立 |
| `attribwrangle5` | attribwrangle | 完全孤立 |
| `merge12` | merge | 完全孤立 |
| `extractpointfromcurve1` | extractpointfromcurve | 完全孤立 |

### 布局优化
- 已自动布局/obj层级节点
- 已自动布局/obj/particle内部321个节点

### 整理统计
- 原始节点数: 338 → 整理后: 321
- 删除孤立节点: 17个

---

## 6. 备注

- 工程使用相机可见性剔除优化粒子数量
- 狗区域使用pointdeform进行粒子变形
- 线条使用trail+carve实现生长动画
- 保留的调试分支（有输入但输出未使用）可根据需要进一步清理
