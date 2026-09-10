---
title: OS-CH7-補充加強觀念
categories:
  - 研究所題目
  - 作業系統
  - 第七章
tags:
  - 作業系統
abbrlink: 5bbc732a
mathjax: true
date: '2026-09-2 14:49:00'
---

- CPU產生的是Logical Address，Memory -> Physical Address
  - Memory 真正使用的是Physical Address
  - 中間用MMU轉換
- 對程式來說是看Logical Address，所以如果實際RAM只有1GB，程式也可以是4GB

| Binding 時機     | 位址什麼時候確定 | Logical = Physical？ |
| -------------- | -------- | ------------------- |
| Compile-time   | 編譯時      | ✅                   |
| Load-time      | 載入 RAM 時 | ✅                   |
| Execution-time | 執行過程中才轉換 | ❌                   |

- TLB (Translation Look-Aside Buffer) 本質上是Page Table的高速 **硬體** Cache

- Base = 1200 ， Limit = 350 => 1200 ~ 1549 (limit - 1)
- Two Level Paging ， 需要 `2` 次 M.A. 才能轉成**Physical Address**
  - Outer Page Table
  - Inner Page Table
  - 如果真的要拿資料 需要第3次
- Dynamic Loading 不會需要OS特別的支援
- Dynamic Linking / Shared Library 防止Duplicated code
- Paging 計算
  - Physical Memory = 16 KB
  - Frame Size      = 1 KB
  - Logical Address = 16 bits
    - Frame Size = Page Size = 1 KB = 2^10 Bytes => Offset = 10 bits
    - Page Number + Offset = 16 => Page Number + 10 = 16 => Page Number = 6 bits
    - 因此共有：2^6 = 64 Pages
    - Physical Memory： 16 KB = 2^14 Bytes => 2^14 / 2^10 = 2^4 = 16 Frames
    - Logical Address = 16 bits
        ```
        ┌─────── 6 bits ───────┬──── 10 bits ────┐
        │     Page Number      │      Offset      │
        └──────────────────────┴──────────────────┘
        Physical Address = 14 bits

        ┌── 4 bits ──┬──────── 10 bits ────────┐
        │ Frame No.  │          Offset         │
        └────────────┴─────────────────────────┘
        ```

## TLB Access Time

已知：
- Memory = `150 ns`
- TLB = `10 ns`
- Hit Ratio = `0.9`

### 無 TLB
Page Table + Data：

$$
150 + 150 = 300ns
$$

### TLB Hit

$$
10 + 150 = 160ns
$$

### TLB Miss

$$
10 + 150 + 150 = 310ns
$$

### Effective Access Time（EAT）

$$
EAT = 0.9(160) + 0.1(310) = 175ns
$$

| 情況 | Access Time |
|---|---|
| 無 TLB | $2M$ |
| TLB Hit | $T + M$ |
| TLB Miss | $T + 2M$ |

$$
EAT = h(T+M) + (1-h)(T+2M)
$$

> **重點：Page Table 在 Memory，所以沒有 TLB 時需要存取 Memory 兩次。**

## 上面部分結束 繼續

- 修改Base , linit 是 **Priviledge instruction** ，所以不能在**User Mode**載入
- Two-level Page Table => 3次**記憶體**存取
  - Logical Address -> Outer Page Table -> Inner Page Table -> Actual Data **共三次**

- Page Table Size = Page Table Entry 數量 * 每個Page Table Entry大小
  - 沒有算Page Size，不存Page的資料
- multi level Page Table ，越多層Page Table有，TLB miss penalty 越高

- Page Fault 大致流程
  - Trap OS
  - 儲存Register,Process State
  - 決定中斷來自 illegal memory Access
  - 確定是不是Page Fault 引起，找Free Frame ，或是Page Replacement
  - Disk I/O Read
  - **當在等待I/O  完成時，CPU 可能可以配置CPU 給其他Process** (不一定發生)
  - 收到I/O 中斷
  - **儲存Register and Process State for other process** (不一定發生)
  - 決定中斷來自Disk
  - OS 修改Page Table
  - CPU 給Page Fault 的Process
  - Restore

- encompass 涵蓋
- Page Fault 時間包含
  - 處理Page Fault Trap
  - 把需要的Page 從 Disk 讀進Memory
  - Restart
- 如果Page在I/O ， 需要把它Lock/in memory
- Valid/invalid bits : invalid 可能有兩種 
  1. Page不屬於此Process的Logical address Space
  2. Page合法但不再Main Memory
- **Associative memory** 可以看成 TLB ， 因為TLB是此技術的一種
- FIFO ， LRU 沒有永遠誰比較好
- **NRU(Not Recently Used)** : 比較優先順序 (R,M)
  - 先分類，從最低類別選 (Reference bits, Modified bits) => (0,0)
  - Enhanced Sencond Chance : 沿著**Circular Queue**逐一掃瞄，有**Clock Pointer**
- Page Table Entries = Virual Address Space / Page Size
- Page 越大 -> Page 數量越少 -> Page Table 越小

- 只讓程式一部份放在Memory中，有甚麼好處
  - 程式可以比RAM大
  - RAM可以容納更多Process，CPU不容易閒著
  - 一開始不用把整個Program 搬進RAM，只載入需要的Pages，所以I/O減少

- Modify bit可以減少Page Fault處理時間，因為可以避免沒修改的Page也寫回Disk
- Thrashing 會導致
  - Paging Disk 幾乎都在工作
  - CPU 不一定 100% idle

- Paging Disk 是說放不進Page Table的Page們在Disk的地方
- 增加Page Size **可能改善** Thrashing
  - 一次載入更多附近資料，因為有locality ，所以可能可以剪少P.F.次數

- Under-clocking the CPU 不會直接造成Thrashing，而是讓執行程式變慢
- **Page File(Swap/Partition)** 就是OS在硬碟上拿來當虛擬記憶體的後援空檢檔案，類似**硬碟上用來放Page的空間**
- 資料若被存取 很快又被存取 : **Temporal Locality** ， 沒有 Spatial 
- memory protection violation is raised by **MMU**
- Swap Space 是Disk 拿來當**RAM的後援空間**
- Address Binding 只有 
  - **Compile Time**
  - **Load Time**
  - **Execution Time**

- 動態載入的Kernel Module，也可以再不需要的時候被卸載
- Kernel 維護一張 不同**Executtable binary Format對應函數**的表，當要載入Executable時，會嘗試適合的Loader
- Load是需要時才會載入
- OS 替每一個Process都維護一份
  - Page Table
  - PTBR (Page Table Base Register) 
  - PCB
  - ~~TLB~~ 不會都維護一份

- **WSS** (Working Set Size)拿來Approximate working Set

- PFF = Page Fault Frequency
- LFU = Least Frequently Used

- 一個Process至少要多少Page Frames，**取決於一條Instruction**，在最壞情況下可能需要同時存取多少個Pages
  - ISA 決定的

- 保證**Stack Algorithm** 的就兩個
  - LRU
  - OPT/MIN
  - 其他不保證
- 不同Virtual Address 可以映射到同一個Physical Memory
- 每個Process 有自己的Page Table，所以**權限可以不一樣**
- Shared Memory **可以被 Swap 到DISK**
- 4-way set associative : 一個set 可以放4個TLB Entry

| Algorithm | 怎麼找            |
| --------- | -------------- |
| First Fit | 第一個夠大的         |
| Next Fit  | 從上次位置開始，第一個夠大的 |
| Best Fit  | 夠用中最小的         |
| Worst Fit | 最大的            |


| 選項                                                | 判斷 | 原因                                                      |
| ------------------------------------------------- | -- | ------------------------------------------------------- |
| (A) Second-Chance uses \((R,M)\)                  | ❌  | 一般 Second-Chance 主要只看 Reference bit \(R\)，答案是Enhanced Second Chance               |
| (B) Additional-Reference-Bits can approximate LRU | ✅  | 用多個歷史 reference bits 記錄最近使用情況                           |
| (C) LRU is optimal                                | ❌  | Optimal 是 OPT / MIN，不是 LRU                              |
| (D) Belady's algorithm exhibits Belady anomaly    | ❌  | Belady's optimal algorithm 是 Stack Algorithm，不會 anomaly |

- Belady's algorithm（又稱 Optimal page replacement algorithm，最佳頁面置換演算法 或 Belady's MIN

- indirect addressing : A存的不是A的資料，**存的是真正的資料在哪裡的Address**