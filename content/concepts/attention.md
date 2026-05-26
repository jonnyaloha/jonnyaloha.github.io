---
title: Attention 메커니즘
parent: 개념
nav_order: 1
math: mathjax
---

# Attention 메커니즘
{: .no_toc }

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 개요

Attention은 입력 시퀀스의 서로 다른 위치에 가중치를 부여해 정보를 집계하는 메커니즘입니다. 2017년 *Attention Is All You Need* 논문에서 제안된 Scaled Dot-Product Attention이 Transformer의 핵심을 이룹니다.

{: .note }
> Attention의 본질은 **Query-Key 유사도에 따라 Value를 가중합**하는 것입니다.

## Scaled Dot-Product Attention

Query $Q$, Key $K$, Value $V$가 주어졌을 때:

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

여기서 $d_k$는 Key 벡터의 차원이며, $\sqrt{d_k}$로 나누는 이유는 내적 값이 차원에 비례해 커지는 것을 막아 softmax의 gradient가 소실되지 않도록 하기 위함입니다.

### 인라인 수식 예시

각 토큰 $x_i$에 대해 $q_i = W_Q x_i$, $k_i = W_K x_i$, $v_i = W_V x_i$를 계산합니다.

## Multi-Head Attention

여러 개의 attention을 병렬로 수행한 뒤 결합합니다:

$$
\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O
$$

$$
\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
$$

{: .tip }
> 일반적으로 $h = 8$ 또는 $h = 16$이 사용되며, 각 head의 차원은 $d_{\text{model}} / h$입니다.

## 코드 예시

```python
import torch
import torch.nn.functional as F

def scaled_dot_product_attention(Q, K, V, mask=None):
    d_k = Q.size(-1)
    scores = torch.matmul(Q, K.transpose(-2, -1)) / (d_k ** 0.5)
    if mask is not None:
        scores = scores.masked_fill(mask == 0, float('-inf'))
    attn = F.softmax(scores, dim=-1)
    return torch.matmul(attn, V), attn
```

## 변형들

- **Causal (Masked) Attention**: 미래 토큰을 가리는 마스킹
- **Cross Attention**: Encoder-Decoder 간 attention
- **Flash Attention**: 메모리 효율적인 구현
- **Grouped Query Attention (GQA)**: KV head 수를 줄여 메모리 절약

{: .warning }
> 시퀀스 길이 $n$에 대해 표준 attention의 시간/메모리 복잡도는 $O(n^2)$입니다. 긴 컨텍스트에서는 Flash Attention이나 Linear Attention 변형을 고려하세요.

## 참고문헌

- Vaswani et al., "Attention Is All You Need", 2017
- Dao et al., "FlashAttention", 2022
