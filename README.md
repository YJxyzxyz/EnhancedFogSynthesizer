# EnhancedFogSynthesizer

一个基于物理大气散射模型的 **道路雾气合成工具**。
适用于 **自动驾驶视觉训练数据增强**，可生成浅雾、中雾、浓雾等不同强度的带雾图像。

---

## ✨ 特性

* 基于 **大气散射模型**：

  $$
  I = J \cdot t + A \cdot (1 - t), \quad t = e^{-\beta \cdot d}
  $$

* 支持多种增强效果：

  * 地平线软融合，避免雾气边界突兀
  * 空气光自适应与柔和渐变
  * 雾团随机扰动（Perlin 噪声）
  * 深度递增模糊（远处更模糊）
  * 局部对比度衰减（贴近真实相机）
  * 可控的全局大气幕

* 提供预设雾级别：`light`、`medium`、`heavy`

* 可通过 **能见度 MOR (Meteorological Optical Range)** 直接控制雾浓度

---

## 📦 安装依赖

```bash
pip install opencv-python numpy
# 推荐安装 contrib 版以启用 guidedFilter
pip install opencv-contrib-python
```

---

## 🚀 快速使用

```python
import cv2
from src.augment.fog import EnhancedFogSynthesizer

# 读取原始图像
img = cv2.imread("clean.jpg")

# 创建雾合成器
synth = EnhancedFogSynthesizer(
    level="medium",       # light / medium / heavy
    y_h_ratio=0.42,       # 地平线位置（0~1）
    global_veil=0.08,     # 大气幕强度
    depth_blur_max=4.0    # 远处最大模糊半径
)

# 生成雾图
hazy, meta = synth.synthesize(img)

# 保存结果
cv2.imwrite("hazy.jpg", hazy)
```

---

## ⚙️ 参数说明

| 参数                   | 默认值        | 作用                             |
| -------------------- | ---------- | ------------------------------ |
| `level`              | `"medium"` | 雾强度预设：`light`、`medium`、`heavy` |
| `mor`                | `None`     | 气象能见度（米），优先于 `level`，值越小雾越浓    |
| `y_h_ratio`          | `0.42`     | 地平线高度（0\~1，相对图像高度）             |
| `vanishing_x_ratio`  | `0.5`      | 远点水平位置（0\~1）                   |
| `perlin_scale_ratio` | `0.18`     | 雾团尺度比例，越大越细腻                   |
| `global_veil`        | `0.06`     | 大气幕强度，调高会整体泛白                  |
| `depth_blur_max`     | `3.5`      | 远处最大模糊半径，增强“深雾”感               |
| `sky_boost`          | `1.25`     | 天空区域雾增强因子                      |
| `road_damp`          | `0.9`      | 路面雾衰减因子，值越小路面雾越浓               |

---

## 🔧 调整雾浓度的方法

* **更浓雾**：

  * 使用 `level="heavy"`
  * 或减小 `mor`（如 `mor=100`）
  * 增大 `global_veil`（如 `0.12`）
  * 增大 `depth_blur_max`（如 `6.0`）
  * 调整 `road_damp=0.85`

* **更轻雾**：

  * 使用 `level="light"`
  * 增大 `mor`（如 `mor=800`）
  * 降低 `global_veil`（如 `0.04`）

---

## 📊 元信息输出

`synthesize()` 除了生成带雾图像，还会返回 `meta`，包含：

* `beta_map` : 散射系数分布
* `A_map` : 空气光分布
* `depth` : 近似深度图
* `t` : 传输图
* `y_h` : 地平线位置

这些数据可用于可视化或作为训练标签辅助。

---

## 🛠️ 批量生成工具

在 `tools/fog_batch.py` 中提供批处理脚本：

```bash
python -m tools.fog_batch --input clean_images --output foggy_images --levels light,medium,heavy
```

会自动读取文件夹中的图像并生成不同强度的雾图。
