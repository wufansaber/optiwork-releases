# Optiwork

Windows x64 顺序光学设计工作台，采用 **Standalone Optics 自主优化算法**。直接打开支持的 Zemax 处方、追迹实光线、分析像差并按制造条件优化。可由 Python 或 AI 编写的程序在本机调用。

v1.3.0 可在固定片数下搜索不同结构与玻璃组合，选择光斑或波前优先，并在本机无人值守优化。

**[下载 Windows 便携版（v1.3.0）](https://github.com/wufansaber/optiwork-releases/releases/download/v1.3.0/Optiwork-1.3.0-Windows-x64.zip)** · **[v1.0 配套 Python 客户端与示例](https://github.com/wufansaber/optiwork-releases/releases/download/v1.0.0/Optiwork-1.0-Python.zip)** · **[正式发布页与 SHA-256](https://github.com/wufansaber/optiwork-releases/releases/latest)**

![Optiwork 1.0 中文宣传图](https://github.com/wufansaber/optiwork-releases/releases/download/v1.0.0/Optiwork-1.0-zh-CN.png)

## 复算海报上的 21 片投影物镜

海报中的镜面位置和光线路径由 Optiwork 计算，颜色区分物方视场。该公开演示取自 Optiland 的 UVProjectionLens，原始资料为美国专利 US 5,831,776。演示采用物理光阑瞄准，物方视场取 0、12、24 mm，共 21 片、42 个折射面，工作波长为 248 nm，物方 NA 为 0.133。此例用于验证追迹和分析工作流。镜片外缘为光学通光包络。

Python 下载包的 `uv-projection/` 提供了镜头、248 nm 单波长示例材料和复算说明。将示例 AGF 导入工作目录后，打开 `UVProjection-21.owx` 即可看三维光路。在 FFT MTF 中选择 512 × 512、0–2000 lp/mm，三个视场合计 **617,676 条入瞳光线**，本例全部透过。从 256 × 256 加密至 512 × 512，子午、弧矢 MTF 最大绝对变化小于 0.0037。此处采用标量 FFT 模型。Wavefront RMS 的 32 × 128 采样另可复算 12,288 条瞳面光线。[处方所据专利](https://patents.google.com/patent/US5831776A/en)

## 从 Zemax 继续设计

完整解压 Windows ZIP，运行 `Optiwork.exe`。通过 **File → Open** 直接打开顺序 `.zmx`，可载入独立的 `.mf` 评价函数；保存为 `.owx` 可继续编辑。支持范围以同轴顺序折射系统为主。遇到坐标断点等暂不能计算的特征，导入报告会说明原因。工程依赖的玻璃可从本机导入 AGF 到工作目录的 `Glasscat`。

## 分析和优化

**同一支 21 片镜头的调焦实测。** 从示例处方的近轴像面出发，只允许共同像面在 ±0.10 mm 内移动。引导式调焦返回 **+5.38 µm**，三个等权视场的综合几何 RMS 光斑半径由 **2.994 µm 降至 1.902 µm（−36.5%）**。用每个视场 32 × 128 条光线独立加密重算，三个视场的光斑均减小。镜片曲率与镜组间距固定。这一结果对应本例的单变量有界调焦。

- **实光线与衍射分析**。可按需查看点列图、FFT MTF、OPD 扇图、Zernike、逐面 Seidel、场曲、畸变和色差。FFT MTF 最高支持 1024 × 1024 瞳面网格。采样不足的视场会标出原因。
- **Standalone Optics 优化引擎**。CPU 并行筛选候选处方；局部求解可联动曲率、厚度、气隙和像面，并对受支持的波前目标做加密采样复核。可实时查看 MF 变化，在预览中比较前后结果，再决定是否应用。开启加工审查后可按已设条件指出违规面并停止。在支持的波前筛选工况下可使用 CUDA。
- **本地 Python / AI API**。桌面程序计算光学结果。外部脚本可打开镜头、设置参数、计算、提交优化任务并读取结果。命令行也接受 JSON 批处理。

## 用 Python 计算一张波前图

解压 Python 客户端 ZIP，在 **Tools → Python / AI API** 启动服务。以下脚本使用工作目录中的端点文件和自己的镜头路径。API 独立会话可与桌面同时使用。

要直接复算随包物镜，可从解压目录运行 `python wavefront.py "你的工作目录/Config/api-endpoint.json"`。此命令会将随包的示例 AGF 导入当前工作目录。

```python
from pathlib import Path
from optiwork_api import Client

workspace = Path("OptiworkData").resolve()
client = Client.from_info(workspace / "Config" / "api-endpoint.json")
s = client.new_session()
try:
    s.open(Path("my-lens.zmx").resolve())
    result = s.image_quality("wavefront_map", {"fieldIndex": 0})
    print(result["data"]["computed"])
finally:
    s.close()
```

命令 `Optiwork.exe --api-schema --api-output schema.json` 可导出机器可读的接口说明，供 AI 编写脚本或接入自动化流程。端点文件含当前会话的本地凭据，请保留在自己的电脑上。

## 工程适用范围

公差模块面向同轴球面系统的半径、曲率、厚度和空气间隔。偏心和倾斜装调不在此版本的良率估算范围内。加工图支持单片和胶合双片、中文 GB 与英文 ISO 版式，以及 PDF、PNG 和可编辑 DXF；制造参数由设计方与加工方确认。

公开便携包提供三家厂商的 **CC0 参考玻璃数据**，来源为 [refractiveindex.info 数据库](https://github.com/polyanskiy/refractiveindex.info-database)。需要与已有 Zemax 工程逐波长对照时，请导入该工程使用的准确 AGF 版本。

新建及导入 MF 的逐光线 OPD 使用主光线减采样光线的符号。对多色 `OPDX`，Optiwork 使用自己定义的共同物理倾斜拟合；比较 Zemax MF 时，应同时核对各点参考和权重。软件会对相应的原生复算范围给出说明。
