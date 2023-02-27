---
title: Rotating reference frame
date: 2022-01-08 09:50:00 +/-TTTT
categories: [Note, Math]
tags: [Math]  
math: true
---

## Relating rotating frames to stationary frames

기저벡터 $\hat{i},\,\hat{j}$ 그리고 $\frac{d(\theta)}{dt} = w,\,\theta(t) = wt + \theta_0$ 

$z$축으로 회전한다고 했을시,

$\hat{i}(t) = (cos\theta(t), -sin\theta(t))$  
$\hat{j}(t) = (sin\theta(t), cos\theta(t))$

이 기저벡터들의 시간에 대한 미분은,

$ \frac{d}{dt}\hat{i}(t) = w(-sin\theta(t), -cos\theta(t)) = -w\hat{j}$  
$ \frac{d}{dt}\hat{j}(t) = w(cos\theta(t), -sin\theta(t)) = w\hat{i}$  

곧 $ \frac{d}{dt}\hat{u} = \Omega\times\hat{u}\,\,(\hat{u}= \hat{i}\,or\,\hat{j},\,\Omega = (0, 0, w))$

## Time derivatives in the two frames

$\mathit{f}\,$가 아래와 같을때  

$\mathit{f}(t) = \mathit{f}_1(t)\hat{i}\,+\,\mathit{f}_2(t)\hat{j}+\,\mathit{f}_3(t)\hat{k}$  
(여기서 $\mathit{f_1},\,\mathit{f_2},\,\mathit{f_3}$은 각 기저 상의 좌표)

이를 시간에 대해 미분하면,  

$ \begin{align} \frac{d}{dt}\mathit{f} &= \frac{d\mathit{f_1}}{dt}\hat{i} + \frac{d\hat{i}}{dt}\mathit{f_1} +
\frac{d\mathit{f_2}}{dt}\hat{j} + \frac{d\hat{j}}{dt}\mathit{f_2} +
\frac{d\mathit{f_3}}{dt}\hat{k} + \frac{d\hat{k}}{dt}\mathit{f_3} \\\\\\
&= \frac{d\mathit{f_1}}{dt}\hat{i} + \frac{d\mathit{f_2}}{dt}\hat{j} + \frac{d\mathit{f_3}}{dt}\hat{k} + [\Omega\times(\mathit{f_1}\hat{i}+\mathit{f_2}\hat{j}+\mathit{f_3}\hat{k})] \\\\\\
&= \bigl(\frac{d\mathit{f}}{dt}\bigr)_r + \Omega \times \mathit{f}
\end{align} $  

여기서 $ \bigl(\frac{d\mathit{f}}{dt}\bigr)_r $는 회전 좌표계상의 $\mathit{f}_n$ 변화율을 의미함.  
$\mathit{f}\,$의 길이가 고정이라면 $ \bigl(\frac{d\mathit{f}}{dt}\bigr)_r = 0 $  

## Conclusion

Rotataion frame상의 고정 길이 벡터의 시간에 대한 미분은 그 고정 길이 벡터와 각 속도의 외적이다.    

$$ C = \overrightarrow{p} + R\overrightarrow{r} $$  
$$ \frac{d(C)}{dt} = \overrightarrow{v} + w \times\overrightarrow{r}$$  


## Reference

<https://en.wikipedia.org/wiki/Rotating_reference_frame>