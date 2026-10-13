# 论文复现标准SOP

> 适用范围：CV / VLM / VLA / LLM / Multimodal / Autonomous Driving 等公开论文复现。  
> 核心原则：**先审计、再拆架构；先最小闭环、再扩大规模；训练前先验证；所有结论都留证据。**

---

# 0. 复现目标定义

开始前先确定复现等级，禁止直接进入环境安装。

| Level | 目标 | 完成标准 |
|---|---|---|
| L1 | Inference Reproduction | 官方权重可完成单样本/小批量推理 |
| L2 | Mini Training Reproduction | 小数据完成训练→checkpoint→推理→评估闭环 |
| L3 | Training Pipeline Reproduction | 复现论文训练阶段、loss、freeze、数据流 |
| L4 | Benchmark Reproduction | 完整数据集 + 官方 evaluator + 接近论文表格指标 |

填写：

```Plain Text
论文：
仓库：
目标Level：
最终要复现的Table/Figure：
本次允许使用的GPU：
时间预算：
磁盘预算：
是否离线：
```

---

# 1. 资源审计

## 1.1 建立资源清单

| 资源 | 是否存在 | 来源 | 完整性 | 本地路径 |
|---|---|---|---|---|
| Paper PDF | ☐ | | | |
| README | ☐ | | | |
| Training code | ☐ | | | |
| Inference code | ☐ | | | |
| Evaluation code | ☐ | | | |
| Base checkpoint | ☐ | | | |
| Task checkpoint | ☐ | | | |
| Raw dataset | ☐ | | | |
| Processed dataset | ☐ | | | |
| Extra annotations | ☐ | | | |
| Metric cache | ☐ | | | |
| Custom CUDA ops | ☐ | | | |
| Docker / env file | ☐ | | | |

## 1.2 建立 `REPRODUCTION_GAP.md`

固定回答四个问题：

```Plain Text
1. 论文声称需要什么？
2. 官方代码实际提供什么？
3. 当前缺少什么？
4. 缺失项是否阻止目标Level？
```

如果缺失：

```Plain Text
完整训练标注 / 特殊token / evaluator / metric cache / pretrained submodule
```

必须先决定：

```Plain Text
A. 获取官方资源
B. 自己重建
C. 降级到Mini Reproduction
D. 停止该复现
```

不要在缺资源时直接开始大规模训练。

---

# 2. 论文架构拆解

## 2.1 先读论文，不先读全仓库

阅读顺序：

```Plain Text
Abstract
  ↓
Method Figure
  ↓
Method
  ↓
Loss
  ↓
Training Setup
  ↓
Appendix
  ↓
Evaluation
```

输出以下结构：

```Plain Text
Input
  ↓
Preprocessor / Tokenizer
  ↓
Encoder
  ↓
Backbone
  ↓
Intermediate Representation / Latent
  ├── Auxiliary Branch A → Loss A
  ├── Auxiliary Branch B → Loss B
  └── Main Head          → Loss Main
  ↓
Output
```

## 2.2 建立论文→代码映射

| 论文模块 | Python实现 | 配置项 | 输入 | 输出 | 是否训练 |
|---|---|---|---|---|---|
| Backbone | | | | | |
| Vision Encoder | | | | | |
| Projector | | | | | |
| Decoder/Head | | | | | |
| Auxiliary Branch | | | | | |
| Loss | | | | | |

优先定位：

```Bash
grep -RniE 'class .*Model|def forward|compute_loss|loss|from_pretrained|freeze|requires_grad' .
```

不要从仓库第一行开始通读。

---

# 3. 固定工作目录

推荐所有论文统一：

```Plain Text
/workspace/PAPER_NAME
├── paper
├── src
│   ├── train_repo
│   └── infer_repo
├── models
│   ├── base
│   └── pretrained
├── data
│   ├── raw
│   ├── processed
│   └── demo
├── envs
├── scripts
│   ├── env
│   ├── data
│   ├── train
│   ├── infer
│   └── eval
├── configs
├── outputs
│   ├── smoke
│   └── formal
├── logs
├── evaluation
├── evidence
└── docs
```

原则：

```Plain Text
原始数据：只读
处理数据：单独目录
Smoke与Formal：绝不混用
失败checkpoint：明确命名
正式checkpoint：固定路径
```

---

# 4. 环境复现

## 4.1 先确认硬件

```Bash
nvidia-smi
uname -a
cat /etc/os-release
ldd --version | head
```

记录：

```Plain Text
OS：
Kernel：
glibc：
GPU：
GPU Memory：
Driver：
CUDA Driver：
```

## 4.2 创建隔离环境

优先级：

```Plain Text
官方 Docker
  >
官方 conda/requirements
  >
自行构建 conda
```

保存：

```Bash
python --version
pip freeze > evidence/pip_freeze.txt
conda list > evidence/conda_list.txt
```

## 4.3 最小Import Test

```Bash
python - <<'PY'
import torch
print(torch.__version__)
print(torch.version.cuda)
print(torch.cuda.is_available())
PY
```

然后逐个测试关键库：

```Plain Text
transformers
deepspeed
flash_attn
xformers
custom ops
```

---

# 5. 数据Contract验证

任何训练前都先回答：

```Plain Text
Dataset一条sample长什么样？
每个字段叫什么？
shape是什么？
dtype是什么？
路径是相对还是绝对？
训练代码实际读取哪些字段？
```

制作一条样本检查：

```Python
sample = dataset[0]
print(sample.keys())
for k, v in sample.items():
    print(k, type(v))
```

重点检查：

```Plain Text
images/video
text/prompt
labels
trajectory/action
mask
latent label
auxiliary label
metadata
```

如果论文需要额外离线标注，先确认这些字段真实存在。

---

# 6. 模型加载验证

依次完成：

```Plain Text
1. Config load
2. Tokenizer/Processor load
3. Model load CPU
4. Model load GPU
5. Single-sample preprocess
6. Single forward
```

检查：

```Plain Text
unexpected keys
missing keys
shape mismatch
dtype mismatch
NaN/Inf
```

不要看到 warning 就立即改 checkpoint；先区分：

```Plain Text
预期的extra module warning
vs
主模型真正缺权重
```

---

# 7. 训练标准流程

永远按：

```Plain Text
Import Test
    ↓
Model Load Test
    ↓
1 Sample Forward
    ↓
1 Sample Backward
    ↓
1 GPU 1~5 Step Smoke
    ↓
Multi-GPU 5 Step Smoke
    ↓
Short Stability Test
    ↓
Formal Training
```

## 7.1 单样本Forward

检查：

```Plain Text
loss finite
output shape
labels alignment
GPU memory
```

## 7.2 单样本Backward

检查：

```Plain Text
loss.backward()成功
gradient finite
trainable parameter存在gradient
frozen parameter无gradient
```

## 7.3 Smoke Training

推荐：

```Plain Text
max_steps = 3~5
save_strategy = no
eval_strategy = no
```

目的只检查：

```Plain Text
OOM
NaN
distributed
optimizer
checkpoint-independent forward/backward
```

## 7.4 Stability Test

推荐：

```Plain Text
5~50 steps
```

记录：

```Plain Text
loss trend
aux loss trend
GPU memory
step time
NaN
```

## 7.5 Formal Training

只有前面全部通过后执行。

---

# 8. 多阶段训练模板

每个 Stage 固定写：

```Plain Text
Stage N

目的：
Input checkpoint：
Dataset：
Model type：
Trainable：
Frozen：
Loss：
Loss weight：
LR：
Batch：
Grad accumulation：
Epoch / Steps：
Precision：
Optimizer：
Distributed strategy：
Output checkpoint：
Success criteria：
```

Checkpoint lineage：

```Plain Text
Base
 ↓
Stage0 checkpoint
 ↓
Stage1 checkpoint
 ↓
Stage2 checkpoint
 ↓
Final
```

禁止通过“最新目录”模糊选择模型。

---

# 9. OOM处理顺序

出现 OOM 时不要立即降低所有配置。

按顺序：

```Plain Text
1. nvidia-smi 检查是否有其他进程
2. 确认选中的GPU是否真为空闲
3. 检查per-device batch
4. 开gradient checkpointing
5. 降sequence/image/video长度
6. 增gradient accumulation
7. ZeRO-2 → ZeRO-3 / FSDP
8. optimizer state offload
9. CPU offload
10. 最后才考虑模型缩小
```

每次只修改一个关键变量，并记录前后显存。

---

# 10. NaN处理顺序

```Plain Text
1. 找第一个NaN step
2. 分别打印main/aux loss
3. 检查input是否NaN
4. 检查logits/hidden states
5. 检查optimizer变体
6. 检查BF16/FP16
7. 检查learning rate
8. 开gradient clipping
9. 检查自定义loss除数/empty mask
```

不要继续使用已产生 NaN 的 checkpoint。

---

# 11. 推理验证

顺序：

```Plain Text
1 sample
   ↓
5 samples
   ↓
full mini set
   ↓
full benchmark
```

记录：

```Plain Text
model path
input path
output path
device
generation config
latency
memory
warnings
```

输出必须保留：

```Plain Text
raw output
```

后处理结果另存：

```Plain Text
normalized output
```

禁止覆盖原始模型输出。

---

# 12. 评估分层

固定区分：

## A. Pipeline Correctness

```Plain Text
能否加载
能否推理
100/100是否完成
是否有empty
是否有NaN/Inf
```

## B. Output Validity

```Plain Text
JSON是否合法
shape是否合法
trajectory长度
token格式
```

## C. Task Metrics

```Plain Text
Accuracy
ADE/FDE
IoU
mAP
PDM-score
BLEU/LLM Judge
...
```

## D. Efficiency

```Plain Text
latency
tokens/s
samples/s
GPU memory
training time
```

## E. Paper Comparison

必须回答：

```Plain Text
是不是相同dataset split？
是不是相同evaluator？
是不是相同checkpoint scale？
是不是相同input resolution？
是不是完整benchmark？
```

否则只能写：

```Plain Text
sanity metric / mini reproduction result
```

---

# 13. 证据固化

每次正式实验至少保存：

```Plain Text
final training script
config
environment
training log
trainer_state
final checkpoint path
raw inference
normalized inference
evaluation output
README / summary
```

关键小文件计算：

```Bash
sha256sum file1 file2 file3
```

保存：

```Plain Text
evidence/SHA256SUMS.txt
```

---

# 14. 开发记录

文档建议固定六章：

```Plain Text
# 环境配置
# 数据集下载预处理
# 权重下载
# 模型训练
# 模型评估
# 模型架构
```

其中“模型架构”应至少包含：

```Plain Text
各部分选型与技术栈
完整系统架构图
逐模块详解
训练数据流
推理数据流
Loss
Freeze策略
Checkpoint结构
模块总结
```

---

# 15. 项目结束标准

只有以下全部满足才算闭环：

```Plain Text
☐ 环境可重建
☐ 数据contract明确
☐ 模型可加载
☐ 单样本forward/backward成功
☐ Smoke成功
☐ Formal训练成功
☐ Final checkpoint明确
☐ Inference成功
☐ Raw结果保留
☐ Evaluation成功
☐ 结果与论文范围区分清楚
☐ 脚本/日志/指标已归档
☐ 开发记录完成
```
