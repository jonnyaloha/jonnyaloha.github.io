---
title: test
parent: Conference
nav_order: 2
math: mathjax
---

# Attention 계산

아래는 Softmax를 실시간 계산한 결과입니다:

<link rel="stylesheet" href="https://pyscript.net/releases/2024.1.1/core.css">
<script type="module" src="https://pyscript.net/releases/2024.1.1/core.js"></script>

<script type="py">
import numpy as np

scores = np.array([2.0, 1.0, 0.1])
exp_scores = np.exp(scores)
softmax = exp_scores / exp_scores.sum()

print(f"입력 점수: {scores}")
print(f"Softmax 결과: {softmax.round(3)}")
print(f"합계: {softmax.sum():.3f}")
</script>