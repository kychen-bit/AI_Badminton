# AI 协同开发约束文档

> 本文档面向 AI-native / vibe coding 协同开发，用于约束后续 AI 与开发者持续迭代本项目。当前阶段以 MVP 为优先目标，避免过度设计。

## 1. 项目目标

本项目旨在研发“羽毛球多模态情境分析系统”，通过融合计算机视觉、场地标定与人体姿态估计，对羽毛球比赛中的关键情境进行量化分析。

核心目标：

- 基于高清摄像机视频追踪羽毛球轨迹。
- 结合场地标定参数，将二维图像坐标映射到场地平面坐标。
- 对羽毛球落点进行“界内 / 界外”辅助判定。
- 基于人体姿态估计捕捉运动员关键点。
- 统计运动员跑动距离、移动热区与覆盖范围。
- 生成可用于裁判辅助、训练分析与战术复盘的基础报告。

当前项目应优先完成一个可运行、可验证、可演示的 MVP，而不是一次性构建完整商业系统。

## 2. MVP 定义

MVP 的目标是打通“视频输入 -> 检测分析 -> 坐标映射 -> 可视化输出 -> 简单报告”的最小闭环。

### 2.1 MVP 必须包含

- 视频文件输入。
- 单机场地标定配置读取。
- 羽毛球检测或轨迹点输入。
- 图像坐标到场地平面坐标的透视变换。
- 落点界内 / 界外判断。
- 运动员姿态关键点检测。
- 运动员位置轨迹估算。
- 跑动距离、移动热区、覆盖范围的基础统计。
- 分析结果可视化，包括视频叠加、轨迹图或热力图。
- 基础报告导出，优先使用 JSON / CSV / Markdown。

### 2.2 MVP 暂不包含

- 多机位实时融合。
- 高精度三维重建。
- 正式裁判系统接入。
- 复杂战术语义识别。
- 账户体系、权限系统、云端部署。
- 大规模训练平台。
- 微服务架构。
- 移动端 App。

### 2.3 MVP 验收标准

- 能在本地读取一段羽毛球比赛视频并完成分析。
- 能通过配置文件加载场地角点与实际尺寸。
- 能输出至少一个落点判定结果。
- 能输出至少一名运动员的移动轨迹与基础体能统计。
- 能生成可读的结果文件。
- 关键模块具备最小单元测试或可重复运行的示例脚本。

## 3. 技术栈分析与选型

### 3.1 选型原则

- 优先选择成熟、文档完善、社区活跃的 Python 生态工具。
- 优先实现本地单体应用，降低调试和部署复杂度。
- 优先使用结构化配置与清晰模块边界，方便 AI 后续继续开发。
- 不在 MVP 阶段引入复杂后端、消息队列、分布式训练或云基础设施。

### 3.2 推荐技术栈

| 层级 | 推荐技术 | 用途 | 约束 |
| --- | --- | --- | --- |
| 语言 | Python 3.11+ | 主开发语言 | 避免混用多语言实现核心逻辑 |
| 计算机视觉 | OpenCV | 视频读取、标定、透视变换、绘制叠加 | 坐标变换必须集中封装 |
| 数值计算 | NumPy | 坐标、矩阵、统计计算 | 禁止在核心计算中散落魔法数字 |
| 数据处理 | pandas | 结果表格、CSV 导出 | 仅用于分析结果，不做重型数据平台 |
| 羽毛球检测 | Ultralytics YOLO 或自定义轻量检测器 | 羽毛球检测、轨迹点生成 | MVP 可先支持手动标注/模拟点作为 fallback |
| 姿态估计 | MediaPipe Pose Landmarker | 人体关键点检测 | 封装成可替换模块 |
| 可视化 | Matplotlib / OpenCV overlay | 轨迹图、热力图、视频叠加 | 图表逻辑与业务计算分离 |
| MVP 界面 | Streamlit | 本地演示界面 | 不引入复杂前端框架 |
| 配置 | YAML / JSON | 标定、模型、输入输出路径 | 配置必须可版本化 |
| 测试 | pytest | 单元测试与回归测试 | 坐标映射与判定逻辑必须测试 |
| 代码质量 | ruff + mypy 可选 | 格式、静态检查 | MVP 阶段不因类型覆盖率阻塞开发 |

### 3.3 技术栈结论

MVP 阶段采用 Python 单体项目最合适。OpenCV 负责视频处理、场地标定与透视变换；MediaPipe 负责姿态估计；YOLO 或轻量检测器负责羽毛球检测；Streamlit 用于快速构建本地演示界面。

该组合可以在较低工程复杂度下快速验证核心算法闭环，并保留后续替换检测模型、增加实时处理或扩展 API 服务的空间。

### 3.4 后续可选扩展

仅当 MVP 稳定后再考虑：

- FastAPI：需要对外提供分析 API 时引入。
- SQLite / DuckDB：需要管理多场比赛结果时引入。
- PyTorch 训练流程：需要自训练羽毛球检测模型时引入。
- 多进程 / GPU 队列：需要实时或批量加速时引入。

## 4. 项目目录结构

推荐目录结构如下：

```text
AI_Badminton/
├── README.md
├── AI_DEV_SPEC.md
├── pyproject.toml
├── requirements.txt
├── configs/
│   ├── sample_court.yaml
│   └── sample_pipeline.yaml
├── data/
│   ├── raw/
│   ├── processed/
│   └── samples/
├── outputs/
│   ├── reports/
│   ├── videos/
│   └── figures/
├── src/
│   └── ai_badminton/
│       ├── __init__.py
│       ├── app/
│       │   └── streamlit_app.py
│       ├── calibration/
│       │   ├── court_model.py
│       │   └── homography.py
│       ├── detection/
│       │   ├── shuttle_detector.py
│       │   └── player_detector.py
│       ├── pose/
│       │   └── pose_estimator.py
│       ├── tracking/
│       │   ├── shuttle_tracker.py
│       │   └── player_tracker.py
│       ├── analytics/
│       │   ├── line_call.py
│       │   ├── movement.py
│       │   └── heatmap.py
│       ├── visualization/
│       │   ├── overlay.py
│       │   └── plots.py
│       ├── reporting/
│       │   └── report_writer.py
│       ├── pipeline/
│       │   └── analyze_video.py
│       └── utils/
│           ├── config.py
│           ├── geometry.py
│           └── video_io.py
├── tests/
│   ├── test_homography.py
│   ├── test_line_call.py
│   └── test_movement.py
└── scripts/
    ├── run_demo.py
    └── calibrate_court.py
```

约束：

- `src/ai_badminton/` 是唯一核心源码目录。
- `data/raw/` 放原始视频，不提交大文件到 Git。
- `outputs/` 放生成结果，不作为核心源码依赖。
- `configs/` 中的示例配置应可直接运行 demo。
- 不在根目录堆放临时脚本，实验脚本统一放入 `scripts/` 或 `notebooks/`。

## 5. 模块职责划分

### 5.1 calibration

负责场地模型、相机标定与透视变换。

- 定义羽毛球场地标准尺寸。
- 读取图像中的场地角点。
- 计算 homography 矩阵。
- 提供 `image_to_court()` 与 `court_to_image()` 接口。

禁止：

- 在业务分析模块中重复实现坐标变换。
- 在多个文件中散落场地尺寸常量。

### 5.2 detection

负责从视频帧中检测羽毛球与运动员。

- 羽毛球检测器输出图像坐标、置信度、帧号。
- 运动员检测器输出人体框或中心点。
- 检测器必须通过统一接口返回结构化结果。

MVP 允许：

- 使用预训练模型。
- 使用手动标注点或模拟检测结果作为 fallback。

### 5.3 pose

负责人体关键点估计。

- 输入视频帧或人体区域。
- 输出关键点坐标、可见性、置信度。
- 不直接计算体能指标。

### 5.4 tracking

负责跨帧关联与轨迹生成。

- 羽毛球轨迹应包含帧号、图像坐标、场地坐标、置信度。
- 运动员轨迹应包含帧号、位置点、可选关键点。
- 跟踪失败时必须显式标记缺失，而不是静默填充。

### 5.5 analytics

负责比赛情境与体能指标计算。

- `line_call.py`：判断落点是否界内 / 界外。
- `movement.py`：计算跑动距离、速度、覆盖范围。
- `heatmap.py`：生成运动热区数据。

约束：

- analytics 模块只接收结构化轨迹数据，不直接读取视频。
- 判断逻辑必须可单元测试。

### 5.6 visualization

负责可视化输出。

- 在视频帧上叠加轨迹、关键点、场地线、判定结果。
- 生成轨迹图、热力图和基础统计图。
- 不承载核心业务计算。

### 5.7 reporting

负责报告生成。

- 输出 JSON / CSV / Markdown。
- 报告中记录输入文件、配置文件、模型版本、关键指标。
- 不依赖 UI。

### 5.8 pipeline

负责串联完整分析流程。

- 加载配置。
- 读取视频。
- 调用检测、姿态、跟踪、分析、可视化、报告模块。
- 保持流程清晰，避免把算法细节写入 pipeline。

### 5.9 app

负责 MVP 本地演示界面。

- 上传或选择视频。
- 选择配置。
- 启动分析。
- 展示可视化结果和报告。

Streamlit 页面只做交互编排，不写核心算法。

## 6. 编码规范

### 6.1 基础规范

- 使用 Python 3.11+。
- 函数、变量、模块使用 `snake_case`。
- 类使用 `PascalCase`。
- 常量使用 `UPPER_SNAKE_CASE`。
- 每个模块保持单一职责。
- 单个函数优先控制在 50 行以内。
- 避免全局可变状态。
- 避免隐式读取硬编码路径。

### 6.2 数据结构规范

核心数据应使用 `dataclass` 或 Pydantic 模型表达，例如：

- `VideoFrame`
- `CourtPoint`
- `DetectionResult`
- `TrackPoint`
- `PoseKeypoints`
- `LineCallResult`
- `MovementMetrics`

坐标字段必须明确坐标系：

- `image_x`, `image_y`：图像像素坐标。
- `court_x`, `court_y`：场地平面坐标，单位优先使用米。
- `frame_index`：从 0 开始的帧序号。
- `timestamp_ms`：视频时间戳，单位毫秒。

### 6.3 配置规范

- 所有路径、模型参数、阈值、场地标定参数都应来自配置文件或函数参数。
- 禁止在算法代码中硬编码本地绝对路径。
- 示例配置必须能在 demo 中直接使用。
- 配置文件字段应保持稳定，修改时同步更新文档和测试。

### 6.4 错误处理

- 对缺失视频、无效配置、标定点不足、模型加载失败给出明确错误。
- 不允许用裸 `except` 吞掉异常。
- 对检测不到目标的帧，应返回空结果或缺失标记，而不是伪造结果。

### 6.5 测试规范

MVP 阶段必须优先测试：

- 透视变换输入输出。
- 界内 / 界外判定。
- 跑动距离计算。
- 配置加载。

建议测试命名：

- `test_homography.py`
- `test_line_call.py`
- `test_movement.py`

## 7. AI 行为约束

后续 AI 协同开发必须遵守以下规则。

### 7.1 任务边界

- 优先完成 MVP 闭环。
- 不主动引入复杂架构。
- 不在没有需求的情况下增加登录、云服务、数据库、微服务或前端工程。
- 每次改动应尽量小而完整。

### 7.2 代码修改规则

- 修改前先阅读相关模块和配置。
- 遵循现有目录结构与命名风格。
- 新增功能必须放入职责匹配的模块。
- 不把临时代码写进核心模块。
- 不删除用户已有代码，除非明确要求。
- 不将大模型权重、视频数据、输出文件提交为源码。

### 7.3 技术选择规则

- 默认使用本文件推荐技术栈。
- 引入新依赖前必须说明原因。
- 如果标准库或现有依赖足够，不新增依赖。
- 对检测模型、姿态模型等可替换能力，必须通过接口封装。

### 7.4 输出质量规则

- 代码必须可读、可测试、可复用。
- 核心算法函数应有类型标注。
- 涉及坐标系转换时必须写清输入输出坐标系。
- 生成报告时必须记录关键配置，保证结果可追溯。
- 对不确定结果必须显示置信度或缺失状态。

### 7.5 AI 禁止行为

- 禁止为了“看起来完整”而伪造实验结果。
- 禁止将演示数据当作真实分析结论。
- 禁止把所有逻辑塞进单个脚本。
- 禁止在 UI 层实现核心算法。
- 禁止未经说明替换核心技术栈。
- 禁止无测试地修改坐标映射、边界判定等关键逻辑。

## 8. 开发流程建议

### 8.1 MVP 推荐迭代顺序

1. 建立项目骨架与配置加载。
2. 实现场地模型与透视变换。
3. 实现界内 / 界外判定。
4. 支持读取视频并抽帧。
5. 接入羽毛球检测结果输入，允许先用手动标注或示例点。
6. 接入姿态估计，输出关键点。
7. 生成运动员轨迹与基础体能指标。
8. 生成轨迹图、热力图和基础报告。
9. 增加 Streamlit 本地演示界面。
10. 补充关键测试与 demo 文档。

### 8.2 每次开发的建议流程

- 明确本次目标和非目标。
- 阅读相关 README、配置和模块代码。
- 先实现最小可用逻辑。
- 添加或更新测试。
- 运行最小验证命令。
- 更新文档或示例配置。
- 总结改动、验证结果和剩余风险。

### 8.3 分支与提交建议

如果后续启用 Git：

- `main`：稳定分支。
- `feat/mvp-pipeline`：MVP 主流程。
- `feat/calibration`：场地标定。
- `feat/pose-analysis`：姿态与体能分析。
- `fix/...`：缺陷修复。

提交信息建议：

```text
feat: add court homography mapping
fix: handle missing shuttle detections
test: cover line call boundary cases
docs: update MVP development spec
```

## 9. MVP 配置示例约束

示例配置建议表达以下信息：

```yaml
video:
  input_path: data/samples/demo.mp4
  frame_step: 1

court:
  unit: meter
  image_points:
    top_left: [100, 120]
    top_right: [1180, 130]
    bottom_right: [1240, 700]
    bottom_left: [80, 690]
  court_points:
    top_left: [0.0, 0.0]
    top_right: [6.1, 0.0]
    bottom_right: [6.1, 13.4]
    bottom_left: [0.0, 13.4]

models:
  shuttle_detector: null
  pose_estimator: mediapipe

outputs:
  report_dir: outputs/reports
  video_dir: outputs/videos
  figure_dir: outputs/figures
```

约束：

- 坐标点顺序必须明确。
- 单位必须明确。
- 模型可以为空，但 pipeline 必须能给出清晰提示或使用 fallback。

## 10. 可维护性原则

- 保持模块化：检测、标定、跟踪、分析、可视化、报告相互独立。
- 保持可替换：模型实现可替换，数据结构和接口尽量稳定。
- 保持可测试：坐标映射和判定逻辑不得依赖真实视频才能测试。
- 保持可追溯：每次分析输出应记录输入、配置、模型、时间和版本。
- 保持轻量：MVP 阶段只做支撑闭环所需的最少工程。

## 11. 参考依据

- [OpenCV Homography 官方文档](https://docs.opencv.org/4.x/d9/dab/tutorial_homography.html)：相机标定、平面单应性与透视变换能力适合本项目的场地标定和坐标映射需求。
- [Ultralytics YOLO Tracking 官方文档](https://docs.ultralytics.com/modes/track/)：YOLO 检测与跟踪流程适合作为羽毛球检测模块的候选实现。
- [MediaPipe Pose Landmarker Python 官方文档](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker/python)：Pose Landmarker 可用于人体关键点检测。
- [Streamlit 官方文档](https://docs.streamlit.io/)：适合快速构建 Python 本地数据应用和 MVP 演示界面。

本文档是后续 AI 协同开发的默认约束。若实际需求发生变化，应先更新本文档，再修改项目结构或核心技术路线。
