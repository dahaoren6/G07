# P02 fastText 论文复现与中期进展

## 小组与分工
组号 G7；万燊涵 1120233197、徐铂尧 1120232601、唐源锴 1120232588、齐锴 1120231190。包名 NLP2026_G07.zip。

## 本次完成范围
AG News 单数据集；固定嵌套 1k/5k/10k；unigram 基线与 unigram+bigram 方法；数据规模扩展；三个固定种子 17/42/2026；共 18 个正式条件。dim=10、epoch=5、lr=.25、bucket=10M、softmax、thread=1，无预训练词向量和字符子词。参数事先固定，验证集不用于选优。主结果：10k unigram 87.26 ± 0.08%、bigram 86.99 ± 0.16%；未复现全量论文的正向提升。

## 环境与来源
Windows x64；Python 3.11.5；numpy 1.26.4；scikit-learn 1.3.0；fasttext-wheel 0.9.2（第三方预编译 Windows 分发，非本次编译附件 main）。实际版本见 results/environment.json；附件 ZIP、论文、wheel、二进制扩展及作者脚本哈希在 code/source_provenance.json、environment.json。作者源码不改，自编代码适配 CSV、规范化、固定抽样、参数调用及评价。作者 MIT 许可与脚本见 code/third_party/。

## 从零运行
建议 Python 3.11 64 位，在独立环境安装；不把环境目录打包。
```powershell
python -m venv .venv
.venv/Scripts/Activate.ps1
python -m pip install -r requirements.txt
python code/download_data.py --output data/raw --workers 8
python code/experiment.py prepare --raw data/raw --work data/prepared
python code/experiment.py worker --work data/prepared --size 1000 --ngram 2 --seed 17 --smoke
python code/experiment.py run --work data/prepared
python code/verify_results.py
python code/analyze.py --work data/prepared
```
最后一步重新生成自动案例，固定 ID 的中文解释保存在 code/case_annotations.json。运行 code/apply_case_annotations.py 可恢复解释与 Markdown。下载脚本按固定 commit 与 Git blob 校验，采用分段重试；网络仍需允许访问 GitHub Raw。

预计资源：wordNgrams=2 的 10M×10 float 参数矩阵约 400 MB，再加词典、输出及运行时开销；建议至少 2 GB 可用内存。磁盘用于原始与处理数据约百余 MB，预测证据约 30 MB。模型在内存训练，未保存全部 .bin 权重，可从固定数据、代码、种子重建。

本机正式训练单次约 0.05–0.57 秒，7600 条推理约 0.045–0.064 秒；含进程、读写、评价的全流程更长，通常数十秒到数分钟。下载时间依网络而定，本次发生重试，耗时显著长于训练。不把训练调用时间当作端到端耗时。

## 数字与证据逐条入口
| 内容 | 路径与核验方式 |
|---|---|
| 一页中期记录 | midterm_progress.pdf / .md |
| 5 页正文与独立参考文献页 | report.pdf；可编辑文字 report.md |
| 7 页主讲与 1 页证据 | slides.pptx、slides.pdf；讲稿见 slides_notes.md |
| 各次实际参数 | configs/experiment.json；results/runs/条件/config.json |
| 18 次指标 | results/all_runs.csv |
| 均值、样本标准差 | results/summary.csv、summary.json |
| 同种子配对差 | results/paired_deltas.json |
| 原始运行日志 | results/full_run.log、results/runs/条件/raw.log |
| 每条输出 | results/runs/条件/test.jsonl、validation.jsonl |
| 逐条输出具体示例 | results/runs/n10000_g2_s17/test.jsonl，按 id 检索 |
| 10 个案例 | results/cases.md、cases.json |
| 固定划分 | data/split_ids.csv、manifest.json、README.md |
| 重叠敏感性 | results/overlap_sensitivity.json（统一 7599 条测试） |
| 独立评价 | results/verification.json；scikit-learn 核验 208800 条 |
| 同种子重训 | results/reproducibility.json、reproducibility_raw.log |
| 资源试跑 | results/smoke_setup.log、results/smoke/n1000_g2_s17_smoke/ |
| 失败排查与解决 | results/setup_notes.md |

每条预测含 id、gold、pred、confidence、run_id、config；config.json 与同目录对应。confidence 为模型输出的 top-1 分数，未做概率校准。validation 4000 + test 7600，每次共 11600 条，18 次共 208800 条；试跑与重训不计入 18 次主结果。推理时间先预热后运行 3 遍取中位数，再对种子取均值；标准差是样本标准差，不是置信区间。

## 实验解释边界
本次是缩规模复现，原文表 1 AG 全量 91.5→92.5 不当作本次应得分数。5k 与 10k 的 bigram 均未超 unigram，1k 的配对效应随种子变号。无显著性检验、无跨抽样波动估计、无词级归因；epoch=5 的低数据训练可能不足，仅是后续假设。下一项计划固定 1k bigram 其余条件，只将 epoch=5 改为 25，尚未执行。

## 数据和 AI
CSV 来源、许可限制、固定版本与命令见 data/README.md。无需 API Key、GPU 或付费模型。Codex 辅助编写、运行与整理；数值为本机实际运行，未用生成内容代替结果。
