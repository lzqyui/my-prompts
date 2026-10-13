# repro_checklist

> 每拿到一篇新论文，复制本文件并逐项勾选。  
> 原则：**有一项关键资源不清楚，就不要直接启动正式训练。**

# A. 目标

- [ ] 已确定论文名称/版本
- [ ] 已保存 Paper PDF
- [ ] 已确定复现 Level（L1/L2/L3/L4）
- [ ] 已确定要复现的 Table/Figure/Metric
- [ ] 已确定时间预算
- [ ] 已确定 GPU/CPU/RAM/磁盘预算
- [ ] 已明确是否离线

# B. 资源审计

- [ ] README 已读
- [ ] Training code 存在
- [ ] Inference code 存在
- [ ] Evaluation code 存在
- [ ] Base model 可获得
- [ ] Task checkpoint 可获得
- [ ] Raw dataset 可获得
- [ ] Processed dataset 可获得/可生成
- [ ] Extra annotations 可获得
- [ ] Metric cache 可获得
- [ ] Custom CUDA ops 可编译
- [ ] Docker/requirements/conda 信息明确
- [ ] 已创建 `REPRODUCTION_GAP.md`

# C. 架构拆解

- [ ] 已画输入→输出总图
- [ ] 已确认 Vision Encoder
- [ ] 已确认 Text Encoder/LLM
- [ ] 已确认 Backbone
- [ ] 已确认 Projector/Aligner
- [ ] 已确认 Latent/Intermediate representation
- [ ] 已确认 Auxiliary branches
- [ ] 已确认 Output Head
- [ ] 已确认所有 Loss
- [ ] 已确认 Loss weights
- [ ] 已确认 Freeze strategy
- [ ] 已确认 Training vs Inference 差异
- [ ] 已建立论文模块→代码文件映射

# D. 环境

- [ ] OS 已记录
- [ ] Kernel 已记录
- [ ] glibc 已记录
- [ ] GPU/Driver 已记录
- [ ] Python 已固定
- [ ] PyTorch/CUDA 已固定
- [ ] Transformers 等核心包已固定
- [ ] 自定义包可 import
- [ ] CUDA 可用
- [ ] BF16/FP16 已测试
- [ ] FlashAttention/xFormers 已测试
- [ ] DeepSpeed/FSDP 已测试
- [ ] `pip freeze` 已保存
- [ ] `conda list` 已保存

# E. 数据

- [ ] 单样本 keys 已打印
- [ ] 所有字段含义明确
- [ ] 所有 shape/dtype 明确
- [ ] image/video 路径有效
- [ ] tokenizer/processor 输出正常
- [ ] labels 与 output 对齐
- [ ] mask 正确
- [ ] auxiliary labels 存在
- [ ] train/val/test split 明确
- [ ] 数据数量已核对
- [ ] 原始数据保持只读

# F. 模型加载

- [ ] Config load 成功
- [ ] Tokenizer load 成功
- [ ] Processor load 成功
- [ ] Model load 成功
- [ ] Missing keys 已解释
- [ ] Unexpected keys 已解释
- [ ] 无 shape mismatch
- [ ] 单样本 preprocess 成功
- [ ] 单样本 forward 成功
- [ ] loss finite

# G. Backward / Smoke

- [ ] `loss.backward()` 成功
- [ ] trainable 参数有梯度
- [ ] frozen 参数无梯度
- [ ] 梯度无 NaN/Inf
- [ ] 1 GPU 1~5 step smoke 成功
- [ ] Multi-GPU 5 step smoke 成功
- [ ] GPU memory 已记录
- [ ] step time 已记录
- [ ] 没有其他用户进程占用目标 GPU
- [ ] Stability test 成功

# H. 多阶段训练

对每个 Stage：

- [ ] Input checkpoint 明确
- [ ] Dataset 明确
- [ ] Trainable modules 明确
- [ ] Frozen modules 明确
- [ ] Loss 明确
- [ ] Loss weights 明确
- [ ] LR 明确
- [ ] Batch 明确
- [ ] Grad accumulation 明确
- [ ] Epoch/Steps 明确
- [ ] Precision 明确
- [ ] Optimizer 明确
- [ ] Parallel strategy 明确
- [ ] Output checkpoint 明确
- [ ] 日志保存
- [ ] Final trainer state 检查
- [ ] NaN/OOM/Traceback 检查

# I. OOM处理

- [ ] 先检查 `nvidia-smi`
- [ ] 确认GPU没有被别人占用
- [ ] 降 per-device batch 前先记录原显存
- [ ] Gradient checkpointing 已考虑
- [ ] Grad accumulation 已考虑
- [ ] Sequence/image/video length 已评估
- [ ] ZeRO-3/FSDP 已评估
- [ ] CPU offload 仅作为必要方案
- [ ] 每次只改一个核心变量

# J. NaN处理

- [ ] 已找到第一个 NaN step
- [ ] Main loss / Aux loss 分开检查
- [ ] Input 无 NaN
- [ ] Hidden/logits 无 NaN
- [ ] LR 已检查
- [ ] Optimizer 已检查
- [ ] Gradient clipping 已检查
- [ ] 不使用 NaN checkpoint

# K. 推理

- [ ] 1 sample inference 成功
- [ ] 5 sample inference 成功
- [ ] mini/full inference 成功
- [ ] Raw output 单独保存
- [ ] Generation config 已记录
- [ ] Latency 已记录
- [ ] GPU memory 已记录
- [ ] Warning 已解释
- [ ] Empty output 已统计
- [ ] NaN/Inf 已统计

# L. 评估

- [ ] Pipeline correctness 已统计
- [ ] Output validity 已统计
- [ ] Task metric 已统计
- [ ] Efficiency 已统计
- [ ] Worst cases 已检查
- [ ] Postprocess 规则已明确
- [ ] Raw 与 postprocessed 指标分开
- [ ] 已确认是否使用官方 evaluator
- [ ] 已确认是否能与论文数字直接比较

# M. 证据

- [ ] Final training script 保存
- [ ] Final config 保存
- [ ] Environment 保存
- [ ] Formal training log 保存
- [ ] Final checkpoint 路径保存
- [ ] Raw inference 保存
- [ ] Normalized inference 保存
- [ ] Evaluation JSON/CSV 保存
- [ ] SHA256 保存
- [ ] Failed experiments 有清晰命名
- [ ] 中间 checkpoint 删除前已验证 final checkpoint

# N. 文档

- [ ] 环境配置
- [ ] 数据集下载预处理
- [ ] 权重下载
- [ ] 模型训练
- [ ] 模型评估
- [ ] 各部分选型与技术栈
- [ ] 完整模型架构图
- [ ] 逐模块详解
- [ ] 训练数据流
- [ ] 推理数据流
- [ ] Loss
- [ ] Freeze策略
- [ ] Checkpoint结构
- [ ] 问题与解决
- [ ] 最终复现范围说明

# O. 完成

- [ ] 结果可以从零重新解释
- [ ] Final checkpoint 可加载
- [ ] Final inference 可运行
- [ ] Metrics 可重新计算
- [ ] 没有把 mini result 写成 full benchmark
- [ ] 没有把 postprocessed metric 写成 raw model metric
- [ ] 项目正式归档
