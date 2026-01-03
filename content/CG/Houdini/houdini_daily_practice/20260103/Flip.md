# Houdini FLIP 流体模拟原理

## FLIP 核心概念

FLIP (Fluid-Implicit Particle) 是一种混合型流体模拟方法，结合了粒子系统和网格求解器的优势。

### 基本原理

FLIP 采用**粒子-网格混合方法**：
- **粒子**：携带流体属性（速度、密度、温度等）
- **网格**：用于高效求解压力和不可压缩性约束

### 核心算法流程

1. **Particle to Grid (P2G)**
   - 将粒子的速度信息转移到体素网格
   - 使用权重插值（通常是双线性或三线性插值）
   - 网格点累积周围粒子的贡献

2. **Grid Solve**
   - 在网格上求解 Navier-Stokes 方程
   - 应用压力投影确保不可压缩性
   - 处理边界条件和碰撞
   - 添加外力（重力、风力等）

3. **Grid to Particle (G2P)**
   - 将更新后的网格速度传回粒子
   - **关键差异**：FLIP 只传递速度的**变化量**（delta），而非整个速度
   - 公式：`v_particle_new = v_particle_old + (v_grid_new - v_grid_old)`

4. **Advection**
   - 根据更新后的速度移动粒子位置
   - 使用 RK2 或 RK4 时间积分方案

### FLIP vs PIC

| 方法 | 特点 | 优缺点 |
|------|------|--------|
| **PIC** (Particle-In-Cell) | 直接将网格速度赋给粒子 | 稳定但耗散严重，细节丢失 |
| **FLIP** | 仅传递速度变化量 | 保留细节和能量，但可能产生噪点 |

Houdini 中通常使用 **FLIP/PIC 混合**：
```
final_velocity = flip_weight * v_flip + (1 - flip_weight) * v_pic
```
- `FLIP Weight = 0.95` 时接近纯 FLIP（细节丰富）
- `FLIP Weight = 0.0` 时退化为 PIC（平滑稳定）

## Houdini FLIP 关键参数

### Particle Separation
- 控制粒子密度
- 越小 → 精度越高，计算量越大
- 通常设置为场景单位的 0.1 - 0.05

### Grid Scale
- 体素网格分辨率
- 通常设置为 `Particle Separation × 2`
- 影响压力求解精度

### Substeps
- 每帧的时间细分
- 高速运动或小粒子间距需要更多 substeps
- 防止粒子穿透和数值不稳定

### Viscosity（粘度）
- 控制流体内部摩擦
- 0 = 水（无粘性）
- 高值 = 蜂蜜、岩浆

### Surface Tension（表面张力）
- 模拟液体表面收缩效应
- 对小水滴和飞溅效果重要

## 优化技巧

### 1. 自适应网格
- 使用 `Narrow Band` 仅在流体周围求解
- 节省内存和计算时间

### 2. 粒子重采样
- `Particle Reseeding`：在粒子稀疏区域补充新粒子
- `Particle Jitter`：打破规则排列，减少视觉伪影

### 3. 边界处理
- **Collision Field**：使用 SDF（Signed Distance Field）表示碰撞体
- **Stick on Collision**：控制流体附着行为
- **Friction**：摩擦系数影响边界滑动

### 4. 速度混合策略
```
合适的 FLIP Weight 根据场景调整：
- 飞溅、水花：0.95 - 1.0（高细节）
- 平静水面：0.7 - 0.85（减少噪点）
- 粘稠液体：0.5 - 0.7（更稳定）
```

## 常见问题与解决

### 粒子爆炸
- **原因**：时间步长过大，CFL 条件不满足
- **解决**：增加 Substeps 或减小 Particle Separation

### 表面噪点过多
- **原因**：纯 FLIP 的数值噪声积累
- **解决**：降低 FLIP Weight 至 0.85 左右

### 穿透碰撞体
- **原因**：粒子移动速度超过网格分辨率
- **解决**：增加 Collision Field 分辨率或 Substeps

### 失去体积
- **原因**：粒子重采样不足或边界泄漏
- **解决**：启用 `Particle Reseeding`，检查 Collision SDF

## 进阶主题

### OpenCL 加速
- Houdini FLIP 支持 GPU 加速
- 在 `FLIP Solver` → `Advanced` → `OpenCL` 启用
- 大幅提升压力求解速度（5-10倍）

### 白水系统 (Whitewater)
- 从主流体发射次级粒子模拟泡沫、浪花、水雾
- `Whitewater Solver` 基于速度散度和曲率检测
- 分类：Foam（泡沫）、Spray（水花）、Bubble（气泡）

### 海洋扩展
- `Ocean Spectrum` 生成基础波形
- FLIP 仅模拟局部扰动（船只、碰撞）
- 两者通过 `Ocean Source` 节点混合
