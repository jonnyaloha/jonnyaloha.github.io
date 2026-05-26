---
title: Transformer
parent: 개념
nav_order: 2
math: mathjax
---

# Transformer

Transformer는 Self-Attention만으로 시퀀스를 처리하는 구조로, 현대 LLM의 표준 아키텍처입니다.

## 구조

기본 Transformer 블록은 다음으로 구성됩니다:

1. **Multi-Head Self-Attention**
2. **Position-wise Feed-Forward Network**
3. **Layer Normalization**
4. **Residual Connection**

블록의 연산은 다음과 같이 표현됩니다:

$$
\begin{aligned}
h &= \text{LayerNorm}(x + \text{MHA}(x)) \\
y &= \text{LayerNorm}(h + \text{FFN}(h))
\end{aligned}
$$

## Positional Encoding

Attention 자체는 순서 정보가 없으므로 위치 정보를 별도로 주입해야 합니다. 원 논문의 sinusoidal 인코딩:

$$
PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)
$$

$$
PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)
$$

최근 모델들은 RoPE(Rotary Positional Embedding)나 ALiBi를 주로 사용합니다.
