---
layout: post
title:  "Computerized Adaptive Test and IRT (2)"
category: news
author: "ronfe"
---

What does a test do? The very essence of test is not to select better students to university or else, but to estimate the real ability an examinee owns.

Computerized Adaptive Test (or CAT) successively selects questions for the purpose of maximizing the precision of the exam based on what is known about the examinee from previous questions. If an examinee performs well, he/she will then be presented with a more difficult question; otherwise, if he/she performs poorly, a simpler question will be presented.

To achieve this goal, the basic CAT is an iterative algorithm with following,

1 Search the questions pool for the optimal one, based on the current estimate of the examinee's ability
2 Present the chosen question to the examinee, who then answers it correctly or incorrectly
3 Update the estimation based on all prior answers
4 Repeat the step 1-3 using new estimation

As a result, different examinees would receive quite different test questions because of different ability estimations. The estimation could be done with Item Response Theory (IRT). IRT is also the preferred methodology for selecting optimal items.
