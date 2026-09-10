---
title: OS-CH5-補充加強觀念
categories:
  - 研究所題目
  - 作業系統
  - 第五章
tags:
  - 作業系統
abbrlink: 917205a6
mathjax: true
date: '2026-08-25 14:49:00'
---
- **Single** Resource RAG : Cycle <=> Deadlock
> 但是多個Resources **不一定**

- DeadLock 要成立 要四個條件達成
    - Mutual Exclusion
    - Hold and wait
    - No Preemptive
    - Circuler Wait

- Deadlock Prevention 通常不會去破壞 **Mutual Exclusion**
    - 因為有些資源天生就不能給兩個使用
- 甚麼時候Run DeadLock Detection
    - **隔一段時間**
    - 系統回應變慢
    - 事件等待太久
    - CPU utilization 太低
- Deadlock Prevention 
  - ~~定期檢查wait-for Graph~~ -> 這個不是Prevention
  - ~~每次分配前都先確認是不是Safe State~~
- DeadLock Safe/Unsafe 是用在 DeadLock Avoidance 不是 Prevention
- DeadLock 4個條件是 **必要條件**
  - 必要條件 : B 要發生時 A一定要存在 : B => A
  - 充分條件 : 有A一定可以保證B發生 : A => B
  - 充分且必要 : A <=> B
- User Level Process　不能自己改自己的Page Table
- SRTF 實務上無法做出來
- DeadLock 四個條件都發生 不一定會DeadLock，有可能他是多Instance
