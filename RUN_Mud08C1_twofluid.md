# Mud08C1_twofluid.py 最简单的运行方式

水沙两相流柱体坍塌算例（`TwoFluidSediment` 模型，FLIP 主导投影），CPU 后端、32 线程。

> 脚本位置在本仓库**根目录**（不是 `example/` 下），运行命令里 `python Mud08C1_twofluid.py` 不带子路径。

---

## 一行命令

在 WSL Ubuntu 终端执行：

```bash
cd ~/work/GeoTaichi8.20-copilot-replicate-mpm-sph-model/GeoTaichi8.20-copilot-replicate-mpm-sph-model && PYTHONPATH=$(pwd) ~/miniconda3/envs/geotaichi/bin/python Mud08C1_twofluid.py
```

就这一条，其余都不用管。运行结束自动打印后处理结果表。

---

## 三个前提（通常已具备，仅作说明）

1. **conda 环境必须是 `geotaichi`**（不是 `geotaichi_mud08c1`）。
   - 原因：本项目的 `src/sdf/BasicShape.py` 需要 `trimesh.interfaces.gmsh`，只有 `geotaichi` 环境（trimesh 3.23.5）有该接口；`geotaichi_mud08c1`（trimesh 4.12.2）已移除，会报
     `ImportError: cannot import name 'gmsh' from 'trimesh.interfaces'`。
   - 验证：
     ```bash
     ~/miniconda3/envs/geotaichi/bin/python -c "import taichi,trimesh; print(taichi.__version__, trimesh.__version__)"
     ```

2. **`PYTHONPATH` 指向仓库根目录**：`geotaichi` 是本地包（未 pip 安装），命令里的 `PYTHONPATH=$(pwd)` 已自动处理。

3. **在仓库根目录下运行**：脚本用相对路径 `mud_twofluid_08C1` 保存输出，`cd` 到根目录即可保证落点正确。

---

## 脚本当前参数（已设好，无需改动）

| 项 | 值 |
|---|---|
| 线程 | `cpu_max_num_threads=32`（本机 16 物理核 / 32 逻辑核，满线程） |
| 时间步 | `Timestep=4e-5` |
| 总时间 | `SimulationTime=2.0` s |
| 保存间隔 | `SaveInterval=0.02` s（共 101 帧） |

---

## 运行与结果

- **耗时**：约 15 分钟（50000 步）。
- **输出目录**：仓库根目录 `mud_twofluid_08C1/`（`particles/`、`grids/`、`vtks/`）。
- **日志**：终端实时打印，同时写入仓库根目录 `day2026_08_20.log`。

结束后的后处理输出示例（论文 5.3 节量纲）：

```
Frames:101
 t[s]    Z/L0    w_c/omega_s   B/L0   alpha_s^max   m_s error
 0.000    0.00        0.000    0.78       0.6060    0.00e+00
 ...
 1.000    6.10        1.094    1.69       0.5797    4.23e-14
 ...
 2.000   13.34        1.029    3.08       0.2824   -3.00e-13
```

| 字段 | 含义 | 正常值 |
|------|------|--------|
| `Frames` | 帧总数 | 101（= 2.0 s / 0.02 s + 初始帧） |
| `w_c/ω_s` | 归一化沉降速度 | ≈ 1.0（全程稳定，复现论文目标） |
| `Z/L0` | 云团下沉距离 | 单调增长（0 → 13.34） |
| `B/L0` | 云团宽度 | 单调铺展（0.78 → 3.08） |
| `m_s error` | 泥沙质量相对误差 | 1e-13 量级（机器精度） |

---

## 后台运行（可选）

不想占用终端时：

```bash
cd ~/work/GeoTaichi8.20-copilot-replicate-mpm-sph-model/GeoTaichi8.20-copilot-replicate-mpm-sph-model && PYTHONPATH=$(pwd) nohup ~/miniconda3/envs/geotaichi/bin/python Mud08C1_twofluid.py > run_mud08c1_twofluid.log 2>&1 &
```

监控帧数进度（每 0.02 s 一帧，最终 101 帧）：

```bash
ls mud_twofluid_08C1/particles/ | wc -l
```
