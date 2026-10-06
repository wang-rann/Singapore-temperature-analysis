# 新加坡未来气温预测及温度与降雨的关系分析

**Singapore Temperature Forecasting & Rainfall Relationship Analysis (1960–2024)**

基于 1960–2024 年新加坡年度气象数据，使用时间序列方法完成两件事：**预测未来气温**，并**检验降雨量能否解释或改善气温预测**。

![Python](https://img.shields.io/badge/Python-3.x-blue) ![statsmodels](https://img.shields.io/badge/statsmodels-time%20series-green) ![Status](https://img.shields.io/badge/type-课程项目-lightgrey)

---

## 1. 项目背景与问题

新加坡位于赤道，受城市热岛效应影响，夜间升温明显。本项目以“**年最低日均气温**”（下文简称“温度”）为研究对象，原因有三：

- 热带城市夜间升温快于白天，更能灵敏反映长期变暖趋势；
- 城市热岛使白天储存的热量夜间缓慢释放，使最低气温上升更明显；
- 最低气温受短期极端天气影响小，序列更平稳，利于建模。

**要回答的问题**

1. 新加坡温度的长期趋势如何？未来几年（2025–2030）大约是多少？
2. 年总降雨量与温度之间是否存在相关关系或滞后影响？
3. 把降雨量作为解释变量加入模型，能否提高预测精度？

## 2. 数据

| 项目 | 说明 |
|---|---|
| 时间范围 | 1960–2024，共 65 年，年度数据 |
| 变量 | `Temp_Min`：年最低日均气温（℃）；`Rainfall`：年总降雨量（mm） |
| 数据来源 | 课程提供的数据集（原始来源：新加坡统计局 SingStat）  |
| 数据质量 | 无缺失值，年份连续，无需插值 |
| 训练 / 测试划分 | 训练集 1960–2014（55 年），测试集 2015–2024（10 年） |


## 3. 方法与流程

```
数据预处理 → 描述统计与平稳性检验 → 基准模型 SES → ARIMA 建模与诊断
          → 模型对比与 2025–2030 预测 → 降雨-温度关系分析（相关/CCF）→ ARIMAX 动态回归 → 对比结论
```

| 步骤 | 做法 |
|---|---|
| 预处理 | 缺失值与年份连续性检查，箱线图检测异常值 |
| 平稳性 | ADF 检验、ACF 图 |
| 基准模型 | 简单指数平滑（SES），平滑参数自动优化（α ≈ 0.52） |
| 主模型 | ARIMA：一阶差分（d=1），根据 ACF/PACF 选出 3 个候选模型，按 AIC 与参数显著性择优 |
| 模型诊断 | 残差散点图、ACF/PACF、Q-Q 图、Shapiro-Wilk、Ljung-Box |
| 降雨关系 | 时序对比图、散点图与 Pearson 相关、ACF/PACF、互相关函数（CCF） |
| 动态回归 | ARIMAX：当期及 1–5 阶滞后降雨量作外生变量，误差项 AR(1) |
| 评估指标 | MAPE、MAE、MSE、RMSE（样本外 2015–2024） |

**工具**：Python（pandas、numpy、statsmodels、scipy、scikit-learn、matplotlib、seaborn）

## 4. 主要发现

### 4.1 温度呈显著上升趋势，序列非平稳

- 温度由 1960 年的 23.7℃ 升至 2024 年接近 25.9℃，累计上升约 **2.2℃**。
- ADF 检验统计量 −0.4683，p = 0.8980，序列非平稳；ACF 缓慢衰减，需一阶差分（d = 1）。

<img src="TempMin_(1960-2024).png" alt="温度序列" width="800">

### 4.2 ARIMA(2,1,0) 优于 SES 基准模型

候选模型中 ARIMA(2,1,0) 参数全部显著且 AIC 最小（20.49）；残差呈白噪声、基本服从正态分布。

| 模型 | MAPE (%) | MAE | RMSE |
|---|---|---|---|
| SES（基准） | 1.6963 | 0.4365 | 0.5125 |
| **ARIMA(2,1,0)** | **1.5381** | **0.3961** | **0.4826** |

![SES vs ARIMA](SES_vs_ARIMA210_Forecast_Comparison.png)

### 4.3 2025–2030 预测

用全样本重新拟合 ARIMA(2,1,0)，预测未来 6 年温度约在 **25.6–25.7℃**，高于 2012 年之前的历史水平；预测区间随年份增加而变宽，说明越远期不确定性越大。

| 年份 | 预测值 (℃) | 95% 置信区间 |
|---|---|---|
| 2025 | 25.64 | 25.08 – 26.20 |
| 2027 | 25.74 | 25.01 – 26.46 |
| 2030 | 25.69 | 24.76 – 26.63 |

![未来预测](ARIMA_MinTemp_Future_Forecast_2030.png)

### 4.4 降雨量与温度几乎无关，且不能改善预测

- Pearson 相关系数仅 **−0.0332**；CCF 在 0–24 阶滞后内均未超出 95% 置信区间。
- 降雨量序列接近白噪声，而温度有明显趋势，两者生成机制不同。
- ARIMAX 中仅当期降雨量系数显著（−0.0002，p = 0.021），量级极小，5 个滞后项均不显著。
- 即使在预测时给了 ARIMAX 真实的测试期降雨数据，其误差仍明显更大：

| 模型 | RMSE | MAE |
|---|---|---|
| ARIMA(2,1,0) | **0.4826** | **0.3961** |
| ARIMAX（含降雨量） | 0.9013 | 0.8254 |

![ARIMA vs ARIMAX](ARIMAX_vs_ARIMA_Performance_Comparison.png)

**结论**：新加坡年度最低气温主要由长期趋势驱动，年际降雨的随机波动对气温预测几乎没有贡献。

## 5. 局限与改进方向

- 年度数据样本量小（仅 65 个观测值），测试集仅 10 年，评估结果有一定随机性。
- ARIMA 预测随时间趋于平缓，对近年加速升温的刻画有限。
- 降雨量为年总量，可能掩盖季节或月度层面的关系；后续可改用月度数据。
- 可引入更具物理意义的外生变量，如全球海表温度异常、厄尔尼诺/拉尼娜指数（ENSO）、城市热岛强度指标等。

## 6. 仓库结构

```
├── README.md
├── code.ipynb                                  # 完整分析代码
├── <img src="TempMin_(1960-2024).png" alt="温度序列" width="800">           # 以下为输出图表
├── SES_vs_ARIMA210_Forecast_Comparison.png
├── ARIMA_MinTemp_Future_Forecast_2030.png
├── ARIMAX_vs_ARIMA_Performance_Comparison.png
├── ARIMA210_FullRange_Backtest_Plot.png
└──report.docx                                   # 完整分析报告（含图表解读）
```

## 7. 如何运行

1. 克隆仓库并安装依赖：
```bash
   git clone https://github.com/w15034923897-cyber/Singapore-temperature-analysis.git
   cd Singapore-temperature-analysis
   pip install pandas numpy scipy statsmodels scikit-learn matplotlib seaborn jupyter
```
2. 数据由课程提供，因版权原因未上传 。
3. 运行 `code.ipynb`，图表会自动保存到输出文件夹。
```bash
git clone https://github.com/wang-rann/Singapore-temperature-analysis.git
cd Singapore-temperature-analysis
pip install pandas numpy scipy statsmodels scikit-learn matplotlib seaborn jupyter
jupyter notebook code.ipynb
```

运行前请将数据文件放在与 notebook 同一目录。图表会自动保存到输出文件夹中。

## 8. 说明

本项目为课程项目，仅用于学习与展示分析方法。

**联系方式**：wangran002@suss.edu.sg ｜ 【LinkedIn / 个人主页】
