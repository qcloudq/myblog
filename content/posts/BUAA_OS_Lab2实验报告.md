---
title: "BUAA_OS_Lab2实验报告"
date: 2022-05-05T07:08:23+08:00
draft: false
---
# Lab2 实验报告



## 01 实验思考题

### Thinking 2.1

**请你根据上述说明，回答问题：**

- **在我们编写的程序中，指针变量中存储的地址是虚拟地址还是物理地址？**

  答：虚拟地址。

- **MIPS 汇编程序中lw, sw使用的是虚拟地址还是物理地址？**

  答：虚拟地址。

---

### Thinking 2.2

- **请从可重用性的角度，阐述用宏来实现链表的好处。**

  答：由于C语言并不支持泛型，所以我们通过宏来实现链表，使其具有”泛型“的特点。这样所有的链表都可以使用一套宏，实现了代码的复用，提高了可重用性。

- **请你查看实验环境中的 /usr/include/sys/queue.h，了解其中单向链表与循环链表的实现，比较它们与本实验中使用的双向链表，分析三者在插入与删除操作上的性能差异。**

  答：在 queue.h 中可以看到：

  ```c
  /* 双向链表
   * A list is headed by a single forward pointer(or an array of forward
   * pointers for a hash table header). The elements are doubly linked
   * so that an arbitrary element can be removed without a need to
   * traverse the list. New elements can be added to the list before
   * or after an existing element or at the head of the list. A list
   * may only be traversed in the forward direction.
  */ 
  ```

  ```c
  /* 单向链表
   * A singly-linked list is headed by a single forward pointer. The
   * elements are singly linked for minimum space and pointer manipulation
   * overhead at the expense of O(n) removal for arbitrary elements. New
   * elements can be added to the list after an existing element or at the
   * head of the list.  Elements being removed from the head of the list
   * should use the explicit macro for this purpose for optimum
   * efficiency. A singly-linked list may only be traversed in the forward
   * direction.  Singly-linked lists are ideal for applications with large
   * datasets and few or no removals or for implementing a LIFO queue.
  */ 
  ```

  ```c
  /* 循环链表
   * A circle queue is headed by a pair of pointers, one to the head of the
   * list and the other to the tail of the list. The elements are doubly
   * linked so that an arbitrary element can be removed without a need to
   * traverse the list. New elements can be added to the list before or after
   * an existing element, at the head of the list, or at the end of the list.
   * A circle queue may be traversed in either direction, but has a more
   * complex end of list detection.
  */ 
  ```

  **双向链表 List：**

  - 插入：List 的插入操作提供了 `LIST_INSERT_AFTER`、`LIST_INSERT_BEFORE` 和 `LIST_INSERT_HEAD` ，都不需要遍历，时间复杂度均为 O(1)。但由于其只有一个指向头部的指针，所以如果想要在尾部插入一个元素，需要遍历链表，时间复杂度为O(n)。
  - 删除：List 的删除操作为 `LIST_REMOVE` ，时间复杂度为 O(1)。

  **单向链表 Singly-linked List：**

  - 插入：Singly-linked List 的插入操作只提供了 `SLIST_INSERT_AFTER`、`SLIST_INSERT_HEAD` ，时间复杂度均为O(1)。由于单向链表的元素只有一个指向后一个元素的指针，所以如果想要在前面插入一个元素，需要遍历链表，时间复杂度为O(n)。
  - 删除：Singly-linked List 的删除操作有 `SLIST_REMOVE_HEAD` 、`SLIST_REMOVE` 。 `SLIST_REMOVE_HEAD` 的时间复杂度为 O(1)，因为单向链表有一个指向头部的指针。而 `SLIST_REMOVE` 的时间复杂度为 O(n)。由于单向链表结构占用空间比双向链表小，所以单向链表适合数据量非常大而几乎不需要删除数据的场合。

  **循环链表 Circular queue：**

  - 插入：Circular queue 的插入操作提供了 `CIRCLEQ_INSERT_AFTER`、`CIRCLEQ_INSERT_BEFORE`、`CIRCLEQ_INSERT_HEAD`、`CIRCLEQ_INSERT_TAIL` 。循环链表具有一个指向尾部的指针，所以可以直接在尾部插入元素。同时，由于其多储存了一个指向尾部的指针，所以占用空间比双向链表多。循环链表的插入操作时间复杂度均为 O(1)。
  - 删除：Circular queue 的删除操作为 `CIRCLEQ_REMOVE` ，时间复杂度为 O(1)。

---

### Thinking 2.3

**请阅读 `include/queue.h` 以及 `include/pmap.h`, 将 `Page_list` 的结构梳理清楚，选择正确的展开结构。**

答：正确的展开结构为 **C** 。

```c
C:
struct Page_list{
    struct {
        struct {
            struct Page *le_next;
            struct Page **le_prev;
        } pp_link;
        u_short pp_ref;
    }* lh_first;
}
```

---

### Thinking 2.4

**请你寻找上述两个 boot_\* 函数在何处被调用。**

答：`boot_pgdir_walk` 在 `boot_map_segment` 中被调用，用于在建立虚拟地址区间到物理地址区间的映射时，根据虚拟地址返回对应的二级页表项。

​	`boot_map_segment` 在 `mips_vm_init` 中被调用，用于将 `UPAGES` 和 `UENVS` 映射到对应的物理地址区间。

---

### Thinking 2.5

**请你思考下述两个问题：**

- **请阅读上面有关 R3000-TLB 的叙述，从虚拟内存的实现角度，阐述 ASID 的必要性**

  Linux 内核给每个进程都提供了一个独立的虚拟地址空间，而同一虚拟地址在不同的地址空间中通常映射到不同的物理地址，所以需要 ASID 来帮助区分不同的地址空间，以免映射到错误的物理地址。

- **请阅读《IDT R30xx Family Software Reference Manual》的 Chapter 6，结合 ASID 段的位数，说明 R3000 中可容纳不同的地址空间的最大数量**

  ![](https://s3.bmp.ovh/imgs/2022/05/05/359c8c9c8af2dba0.png)

  `EntryHi` 的位结构如上图，可以看到 ASID 段有 **6** 位。

  > By setting up TLB entries with a particular ASID setting and with the EntryLo G bit zero, those entries will only ever match a program address when the CPU’s ASID register is set the same. This allows software to map up to **64 different address spaces** simultaneously, without requiring that the OS clear out the TLB on a context change.
  >
  > ----《IDT R30xx Family Software Reference Manual》

  所以 R3000 中可容纳不同地址空间的最大数量为 **64** 个。

---

### Thinking 2.6

**请你完成如下三个任务：** 

- **tlb_invalidate 和 tlb_out 的调用关系是怎样的？**

  `tlb_invalidate` 调用了 `tlb_out` 。

- **请用一句话概括 tlb_invalidate 的作用**

  更新 TLB 。

- **逐行解释 tlb_out 中的汇编代码**

  ```c
  LEAF(tlb_out)
  //1: j 1b
  nop
      mfc0    k1,CP0_ENTRYHI   //将EntryHi寄存器的值存入k1寄存器中
      mtc0    a0,CP0_ENTRYHI   //将a0寄存器的值存入EntryHi寄存器中
      nop
      tlbp    //根据EntryHi中的值，查找TLB中与之对应的表项
      		//并将表项的索引存入Index寄存器
      		//若未找到匹配项，则Index最高位被置1
      nop
      nop
      nop
      nop
      mfc0    k0,CP0_INDEX     //将Index寄存器的值存入k0寄存器中
      bltz    k0,NOFOUND		 //如果k0的值小于0，即没有找到匹配项
      						 //跳转到NOFOUND
      nop
      mtc0    zero,CP0_ENTRYHI  //将EntryHi寄存器置0
      mtc0    zero,CP0_ENTRYLO0 //将EntryLo寄存器置0
      nop
      tlbwi   //以Index寄存器中的值为索引,将EntryHi和EntryLo的值写到对应TLB表项中
  NOFOUND:
  
      mtc0    k1,CP0_ENTRYHI  //将k1寄存器的值存入EntryHi寄存器中
  							//即恢复EntryHi初始状态
      j   ra  //返回
      nop
  END(tlb_out)
  ```

---

### Thinking 2.7

**在现代的 64 位系统中，提供了 64 位的字长，但实际上不是 64 位页式存储系统。假设在 64 位系统中采用三级页表机制，页面大小 4KB。由于 64 位系统中字长为 8B，且页目录也占用一页，因此页目录中有 512 个页目录项，因此每级页表都需要 9 位。因此在 64 位系统下，总共需要 3 × 9 + 12 = 39 位就可以实现三级页表机制，并不需要 64 位。现考虑上述 39 位的三级页式存储系统，虚拟地址空间为 512 GB，若记三级页表的基地址为 PTbase ，请你计算：**

- **三级页表页目录的基地址**
  $$
  PD_{base}=PT_{base}|PT_{base}\gg 9|PT_{base}\gg 18
  $$

- **映射到页目录自身的页目录项(自映射)**

$$
PDE_{self-mapping}=PT_{base}|PT_{base}\gg 9|PT_{base}\gg 18|PT_{base}\gg 27
$$

---

### Thinking 2.8

- **简单了解并叙述 X86 体系结构中的内存管理机制，比较 X86 和 MIPS 在内存管理上的区别。**

  - X86 架构中对内存的管理使用两种方式，即分段和分页。

    在 X86 架构中内存被分为三种形式，分别是逻辑地址，线性地址和物理地址。如下图所示，通过分段可以将逻辑地址转换为线性地址，而通过分页可以将线性地址转换为物理地址。

    ![](https://s3.bmp.ovh/imgs/2022/05/05/bb45af700757f0ae.png)

  - MIPS 架构对内存使用分页管理。

  - X86 架构采用硬件TLB重填，即由硬件完成页表遍历，将所需的页表项填入TLB中。

  - MIPS 架构采用软件TLB重填，即查找TLB发现不命中时，将触发TLB重填异常，由异常处理程序进行页表遍历并进行TLB填入。

## 02 实验难点

#### 链表宏

本次实验中，对链表宏的理解是一大难题。可以借助助教发的图帮助理解。

![](https://s3.bmp.ovh/imgs/2022/05/05/c266b9262b3a7b4b.jpg)

#### 页目录自映射

页目录自映射也是一大难题。可以借助实验指导书上的图加以理解。

![](https://s3.bmp.ovh/imgs/2022/05/05/d5a6891878ec60af.png)

## 03 体会与感想

​		本次实验我花费了挺长时间来理解概念，恶补理论课知识。在做 Lab 2-2 前，我先把理论课作业做了，里面有一道虚拟地址访存题，帮助我很好地理解了两级页表的结构，补充了相关知识来完成第二部分的实验。

​		最痛苦的是理解页目录自映射，我可能脑子缺了一根筋，怎么都想象不出来是怎么映射的。看理论课ppt也看不懂，网上搜大佬的博客来看也看不懂。直到有一天早上爬起来继续在电脑上看指导书，突然被上面那张指导书上的图击中了，茅塞顿开，醍醐灌顶，福至心灵，理解了它是怎么实现的，所以可能真的要多睡觉（不是）。

