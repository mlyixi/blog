---
title: "pymaBandit说明"
date: 2014-09-02T14:08:52+08:00
tags: ["MAB"]
categories: ["ML"]
toc: true
---

# 分布
* Bernoulli distribution
* Poisson distribution
* Exponential distribution

# 策略
* Gittin's Bayesian optimal strategy for binary rewards [1]
* The classical UCB policy [2]
* The UCB-V policy [3]
* The KL-UCB policy [4]
* The Clopper-Pearson policy for binary rewards [4]
* The MOSS policy [5]
* The DMED policy [6]
* The Emipirical Likelihood UCB [7]
* The Bayes-UCB policy [8]
* The Thompson sampling policy [9]

[1] Bandit Processes and Dynamic Allocation Indices J. C. Gittins. Journal of
the Royal Statistical Society. Series B (Methodological) Vol. 41, No. 2. 1979
pp. 148–177

[2] Finite-time analysis of the multiarmed bandit problem Peter Auer, Nicolò
Cesa-Bianchi and Paul Fischer. Machine Learning 47 2002 pp.235-256

[3] Exploration-exploitation trade-off using variance estimates in multi-armed
bandits J.-Y. Audibert, R. Munos, Cs. Szepesvári.  Theoretical Computer
Science Volume 410 Issue 19 Apr. 2009 pp. 1876-1902

[4] The KL-UCB Algorithm for Bounded Stochastic Bandits and Beyond
A. Garivier, O. Cappé JMLR Workshop and Conference Proceedings Volume 19: COLT
2011 Conference on Learning Theory pp. 359-376 Jul. 2011

[5] Minimax Policies for Adversarial and Stochastic Bandits J-Y. Audibert and
S. Bubeck. Proceedings of the 22nd Annual Conference on Learning Theory 2009

[6] An asymptotically optimal policy for finite support models in the
multiarmed bandit problem J. Honda, A. Takemura. Machine Learning 85(3) 2011
pp. 361-391

[7] UCB policies based on Kullback-Leibler divergence O. Cappé, A. Garivier,
O-A. Maillard, R. Munos. in preparation

[8] On Bayesian Upper Confidence Bounds for Bandit Problems E. Kaufmann,
O. Cappé, A. Garivier.  Fifteenth International Conference on Artificial
Intelligence and Statistics (AISTAT) Apr. 2012

[9] Thompson Sampling: an optimal analysis E. Kaufmann, N. Korda et R.
Munos. preprint

[10] On Upper-Confidence Bound Policies for Non-stationary Bandit Problems
A. Garivier, E. Moulines. Algorithmic Learning Theory (ALT) pp. 174-188
Oct. 2011

[11] Optimism in Reinforcement Learning and Kullback-Leibler Divergence
S. Filippi, O. Cappé, A. Garivier. 48th Annual Allerton Conference on
Communication, Control, and Computing Sep. 2010

[12] Parametric Bandits: The Generalized Linear Case S. Filippi, O. Cappé,
A. Garivier, C. Szepesvari. Neural Information Processing Systems (NIPS)
Dec. 2010