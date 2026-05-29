---
title: test
parent: Conference
nav_order: 2
math: mathjax
---
# Softmax 함수

Softmax는 logits을 확률 분포로 변환하는 함수입니다.

## 수식

$$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_j e^{x_j}}$$

## 실시간 계산 예제

<iframe 
  src="/static/demos/softmax.html" 
  width="100%" 
  height="200" 
  style="border: 1px solid #ccc; border-radius: 6px;"
  loading="lazy">
</iframe>

<iframe src="/static/demos/sigmoid-plot.html" width="100%" height="450" style="border: none;"></iframe>

위 결과는 페이지 열 때 브라우저에서 직접 Python(numpy)을 실행한 거예요.