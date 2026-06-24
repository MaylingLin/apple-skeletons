# Apple Skeletons

Apple Skeletons 是一个面向苹果三维结构重建的研究代码仓库。该项目旨在仅基于普通 RGB 图像恢复苹果的三维结构，不依赖深度相机、激光扫描仪或其他专用三维采集设备，从而降低数据采集成本和设备部署复杂度，并提升在实际农业场景中的使用便利性。

1. 项目目标

本项目面向有限视角条件下的苹果三维重建问题。具体而言，项目从至少两张 RGB 图像中恢复苹果的三维点云，并结合苹果参数化几何模型，对苹果进行形状拟合、不可见区域补全和表面重建，最终得到较完整的苹果三维模型。

与依赖完整多视角扫描或固定实验室采集环境的方法不同，本项目关注在目标仅被部分观测的情况下，如何利用有限视角图像信息与理论几何模型约束，实现对苹果整体三维结构的合理恢复。

2. 核心技术特点

本代码仓主要体现以下技术特点：

1）仅基于 RGB 图像进行重建
项目不依赖深度相机或专用三维扫描设备，只需使用普通相机采集 RGB 图像即可开展三维重建，有助于降低硬件成本和部署门槛。
2）支持有限视角下的三维结构恢复
在实际采集场景中，苹果往往只能从少量视角被观测，部分区域不可见。本项目通过结合可见区域的图像重建结果与苹果参数化几何模型，对不可见区域进行合理补全，从而获得较完整的三维结构。
3）面向实际应用场景设计
该方法不依赖严格固定的实验室采集环境，可适用于传送带分拣、果园户外采摘、移动机器人巡检等实际农业场景，为低成本、快速部署的水果三维感知提供技术支撑。

3. 三维模型的应用价值

基于重建得到的苹果三维模型，可以获取目标物的整体几何形状信息，并形成对其表面空间结构的显式表达。在此基础上，可进一步支持以下应用：

1）苹果表面瑕疵检测与瑕疵区域面积计算；
2）苹果尺寸、形状和表面积等几何信息分析；
3）苹果表面结构分析与质量分级；
4）数字孪生建模与虚拟重建；
5）后续可视化分析、仿真计算与数据管理；
6）机器人采摘过程中的目标识别、空间定位和路径规划分析。

因此，本项目获得的不仅是对苹果尺寸等局部指标的估计能力，而是面向苹果完整三维结构恢复和表面空间表达的建模能力。

4. 方法的通用性与可扩展性

本代码仓的核心技术思想并不依赖苹果这一特定对象，而是基于“理论几何模型约束三维重建过程”的通用框架。

对于苹果，本项目使用苹果参数化几何模型作为形状先验，对有限视角下得到的局部三维结果进行拟合、优化和补全。若将其中的理论几何模型替换为其他水果对应的形状模型，该方法同样可迁移至梨、桃、柑橘等具有相对规则几何特征的水果三维重建任务，体现出一定的跨类别适应性与泛化能力。

需要说明的是，当前代码仓主要以实验脚本形式组织，部分脚本中仍包含本地路径和实验参数。实际运行前，使用者需要根据自身数据存放位置、模型权重路径和实验环境，对相关路径及参数进行相应修改。

## 项目结构

```text
apple-skeletons/
├── apple_model/
│   ├── ideal_model.py              # 参数化苹果模型
│   ├── apple_optimizer.py          # 苹果模型拟合与 CPD 配准
│   └── correspondence_loss.py      # 基于苹果模型配准的对应损失
├── cal_metrics/
│   └── cal_metrics.py              # 网格修复与指标计算工具
├── datasets_preprocess/
│   ├── preprocess_apple.py         # 苹果数据预处理脚本
│   ├── rename_mask.py              # 掩码重命名工具
│   └── preprocess_*.py             # 继承自 DUSt3R 的数据集转换脚本
├── external/
│   ├── dust3r/                     # DUSt3R 依赖代码
│   └── dust3r_visloc/              # DUSt3R 视觉定位相关依赖
├── fill_void/
│   ├── calculate_height_dia.py     # 高度与直径估计
│   ├── surface_recon.py            # 表面重建与体积估计
│   └── void_filling.py             # 点云融合与孔洞补全实验
├── get_scaled_metrics.py           # 预测指标尺度对齐
├── rec_surface.py                  # 表面与缺陷面积估计实验
├── run_criterion.py                # 端到端评估脚本
├── run_inference.py                # 双图像推理示例
├── run_multi_image_inference.py    # 多图像推理示例
├── train.py                        # DUSt3R 训练入口
├── visloc.py                       # 视觉定位入口
├── requirements.txt
└── requirements_optional.txt
```

## 环境安装

克隆仓库并进入项目目录：

```bash
git clone https://github.com/MaylingLin/apple-skeletons.git
cd apple-skeletons
```

建议创建独立 Python 环境，并优先使用支持 CUDA 的环境：

```bash
conda create -n apple-skeletons python=3.10
conda activate apple-skeletons
```

根据自己的 CUDA 版本安装 PyTorch，然后安装项目依赖：

```bash
pip install -r requirements.txt
```

部分网格修复、视觉定位和可选数据读取功能可能还需要额外依赖：

```bash
pip install -r requirements_optional.txt
```

项目中会直接导入 `dust3r`。如果运行时提示找不到 `dust3r`，可以将外部依赖目录加入 `PYTHONPATH`：

```bash
export PYTHONPATH=$PWD/external/dust3r:$PWD/external/dust3r_visloc:$PYTHONPATH
```

## 数据准备

推理脚本需要输入裁剪后的 RGB 图像和对应的苹果前景掩码。一个典型的数据组织方式如下：

```text
data/
└── apple/
    └── Training/
        ├── images/
        │   ├── color_7.jpg
        │   └── color_8.jpg
        └── masks/
            ├── mask_7.npy
            └── mask_8.npy
```

多图像推理通常使用如下结构：

```text
your_data_root/
├── subset/
│   ├── image_000.jpg
│   ├── image_001.jpg
│   └── ...
└── masks_cropped/
    ├── image_000.npy
    ├── image_001.npy
    └── ...
```

其中，掩码文件应为 NumPy 数组，苹果前景区域像素值为 `255`。

`datasets_preprocess/preprocess_apple.py` 提供了一个简单的数据整理脚本，可以将原始苹果数据中的 RGB 图像和深度图复制到训练所需目录。使用前需要修改脚本中的 `root_dir` 等路径。

## 模型权重

请根据实际情况修改脚本中的 `model_name`：

```python
model_name = "output/checkpoint-best.pth"
```


## 快速开始

### 双图像苹果重建

运行前需要修改 `run_inference.py` 中的以下内容：

- `model_name`：苹果微调模型的权重路径。
- `load_images` 中传入的图像路径。
- `mask_root` 和具体掩码文件名。

修改后运行：

```bash
python run_inference.py
```

该脚本会执行双图像三维重建推理、全局点云对齐、基于掩码的苹果前景点提取、参数化苹果模型拟合，并对重建结果进行可视化。

### 多图像苹果重建

运行前需要修改 `run_multi_image_inference.py` 中的以下内容：

- `model_name`：模型权重路径。
- `data_root`：包含 `subset/` 和 `masks_cropped/` 的数据根目录。
- 必要时修改点云或网格输出路径。

修改后运行：

```bash
python run_multi_image_inference.py
```

该脚本会从至少2张苹果图像中重建点云，进行离群点过滤、法向估计、网格重建、苹果模型拟合，并将目标点云与苹果先验模型对齐。

### 端到端指标评估

运行前需要修改 `run_criterion.py` 中的以下内容：

- `model_name`：模型权重路径。
- `data_root_dir`：评估样本所在目录。
- `idx1`、`idx2`：用于重建的图像编号。
- `gt_metrics`：真实高度、直径和体积等指标。

修改后运行：

```bash
python run_criterion.py
```

该脚本会完成苹果重建、模型拟合、生成 `aligned.ply`、表面重建、尺寸与体积估计，并通过尺度对齐将预测指标与真实指标进行比较。

## 训练

训练入口继承自 DUSt3R：

```bash
python train.py --help
```

请先将数据整理为 DUSt3R 支持的训练格式，然后根据 DUSt3R 的参数说明传入训练配置。

## 表面重建与尺寸测量

表面重建功能位于 `fill_void/surface_recon.py`：

```bash
python fill_void/surface_recon.py
```

默认情况下，该脚本会读取 `aligned.ply`，执行点云降采样、离群点过滤、Screened Poisson 表面重建，输出 `tmp_surface.ply`，并估计体积。

高度与直径估计功能位于 `fill_void/calculate_height_dia.py`：

```bash
python fill_void/calculate_height_dia.py
```

该脚本会读取重建后的网格，计算轴对齐包围盒，并返回两个方向的直径和高度。

## 缺陷面积估计

`rec_surface.py` 包含一个实验性的缺陷面积估计流程。脚本会从带颜色的点云或网格中采样表面点，将颜色较暗的区域视为潜在缺陷区域，并结合表面积估算缺陷面积。

运行前需要修改本地网格路径：

```python
mesh_path = "path/to/your/apple_mesh.ply"
```

然后运行：

```bash
python rec_surface.py
```

## 注意事项

- 仓库中部分脚本仍包含本地 Windows 路径，例如 `D:\Projects\...` 或 `C:\Users\...`，运行前需要替换为自己的数据路径。
- 多数脚本会调用 Open3D 可视化窗口，建议在带显示环境的机器上运行。
- `pycpd`、`pymeshlab`、`pyvista`、`pymeshfix` 等依赖在不同系统上可能需要额外安装系统库。
- 当前仓库未包含预训练模型权重和数据集，需要用户自行准备并修改脚本路径。
