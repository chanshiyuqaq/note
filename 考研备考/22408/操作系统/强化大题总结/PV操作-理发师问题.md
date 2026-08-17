
## 一、核心思想

理发师问题本质上是 **"服务-被服务"** 问题，与生产者-消费者问题思路同源：

| 视角 | 角色 | 操作 |
|------|------|------|
| 顾客 → 生产者 | 生产"顾客资源" | `V(customer)` |
| 服务人员 → 消费者 | 消费"顾客资源" | `P(customer)` |
| 服务人员 → 生产者 | 生产"服务人员资源" | `V(server)` |
| 顾客 → 消费者 | 消费"服务人员资源" | `P(server)` |

> **双向同步**：顾客等服务员 ↔ 服务员等顾客，两个信号量各管一个方向。

---

## 二、五大万能模板速查

### 模板总览

| 模板 | 等待上限 | 服务人员行为 | 满员时 | 关键特征 |
|:-----|:--------|:------------|:------|:--------|
| 1 | 无 | 休息（阻塞） | N/A | 最简骨架 |
| 2 | 有 (`waiting` 变量) | 休息 | 离开 | 经典理发师问题 |
| 3 | 有 (`waiting` 变量) | **忙等** | 离开 | 去掉 `P(customer)` |
| 4 | 有 (`waiting` 变量) | 休息 | 离开 | else 分支里 `P(customer)` |
| 5 | 有 (`empty` 信号量) | 休息 | **等待** | `P(empty)` 阻塞等空位 |

---

### 万能模板 1 —— 无等待上限，服务人员可休息（**骨架**）

```
店里有 N 名服务人员，没有顾客时休息，有顾客时叫号。
顾客到店 → 取号 → 唤醒服务员 → 等待叫号 → 被服务。
```

```c
semaphore customer = 0;  // 顾客资源数
semaphore server   = 0;  // 服务人员资源数
semaphore mutex    = 1;  // 互斥锁

Server() {
    while (1) {
        P(customer);      // ① 等顾客（无则阻塞 → 休息）
        P(mutex);         // ② 进临界区
        叫号;
        V(mutex);         // ③ 出临界区
        V(server);        // ④ 通知顾客：我准备好了
        提供服务;          // ⑤ 在临界区外执行
    }
}

Customer() {
    P(mutex);             // ① 进临界区
    取号;
    V(mutex);             // ② 出临界区
    V(customer);          // ③ 唤醒服务人员
    P(server);            // ④ 等服务人员就绪（忙则阻塞）
    被服务;                // ⑤ 在临界区外执行
}
```

---

### 万能模板 2 —— 有等待上限（店满离开），服务人员可休息（方式1）⭐

> 在模板 1 基础上：增加 `waiting` 变量 + `if (waiting < M)` 判断。
> **这是最经典的理发师问题解法。**

```c
semaphore customer = 0;
semaphore server   = 0;
semaphore mutex    = 1;
int waiting        = 0;      // 当前等待的顾客数

Server() {
    while (1) {
        P(customer);         // 没顾客 → 睡觉
        P(mutex);
        叫号;
        waiting--;           // 等候人数 -1
        V(mutex);
        V(server);
        提供服务;
    }
}

Customer() {
    P(mutex);                //准备要叫号了
    if (waiting < CHAIRS) {  // 有空椅子？
        取号;
        waiting++;           // 等候人数 +1
        V(mutex);
        V(customer);         // 唤醒理发师
        P(server);           // 等理发师空闲
        被服务;
    } else {
        V(mutex);            // ⚠️ 别忘释放锁
        离店;                 // 满了，走人
    }
}
```

> **关于 `waiting--` 的时机**：`waiting` 计数的是**等候椅上的人数**，不是"店内总人数"。理发师叫号 → 顾客从等候椅转移到理发椅 → 等候区空出一个位置 → `waiting--`。这个减法是正确且必要的，它让后续顾客能准确判断 `if (waiting < CHAIRS)` 是否有空椅子可坐。


---

### 万能模板 3 —— 有等待上限，服务人员**忙等**

> `waiting > 0` 的判断代替 `P(customer)`，服务员 **不休眠 → 不断轮询**。

```c
Server() {
    while (1) {
        P(mutex);
        if (waiting > 0) {   // 有顾客？
            // P(customer);   ← 去掉了！waiting 是无阻塞版替代
            叫号;
            waiting--;
            V(mutex);
            V(server);
            提供服务;
        } else {
            V(mutex);
            // 什么也不做，下一轮循环（忙等）
        }
    }
}
// Customer() 与模板 2 完全相同
```

---

### 万能模板 4 —— 有等待上限，服务人员可休息（方式2）

> 模板 3 的 else 分支里加 `P(customer)`，**忙等 → 休息**。

```c
Server() {
    while (1) {
        P(mutex);
        if (waiting > 0) {
            叫号;
            waiting--;
            V(mutex);
            V(server);
            提供服务;
        } else {
            V(mutex);
            P(customer);      // 没有顾客，阻塞休息
            // 被唤醒后进入下一轮循环
        }
    }
}
// Customer() 与模板 2 完全相同
```

**模板 2 vs 模板 4 的区别**：

| | `P(customer)` 位置 | 特点 |
|:---|:---|:---|
| 模板 2 | 函数**开头**先 P | 直接阻塞等顾客 |
| 模板 4 | `else` 分支**末尾** P | 先检查 `waiting`，没有才阻塞 |

> 两种等价，推荐用模板 2。

---





### 万能模板 5 —— 无等待上限，服务人员可休息，王道银行问题

> 和模板 1 的区别：满员不离开，用 `empty` 信号量**阻塞等待空位**。

```c
semaphore customer = 0;
semaphore server   = 0;
semaphore empty    = M;      // 空位数
semaphore mutex    = 1;

Server() {
    while (1) {
        P(customer);
        P(mutex);
        叫号;
        V(mutex);
        V(server);
        V(empty);            // 服务完释放一个空位
        提供服务;
    }
}

Customer() {
    P(empty);                // 没空位？阻塞等（不离开！）
    P(mutex);
    取号;
    V(mutex);
    V(customer);
    P(server);
    被服务;
}
```

---

## 三、`waiting` 变量 vs `empty` 信号量

| 方式 | 类型 | 满员行为 | 机制 |
|------|------|----------|------|
| `waiting` + `if` | 普通整型变量 | **离开** | 判断超过上限 → else 分支离开 |
| `empty` 信号量 | 信号量 | **阻塞等待** | `P(empty)` 自动阻塞，服务员 `V(empty)` 唤醒 |

---

## 四、死锁陷阱：P 操作顺序不能换！

**错误写法**（理发师先拿锁再等顾客）：

```c
Server() {
    P(mutex);        // ① 先拿互斥锁
    P(customer);     // ② 没顾客，阻塞！→ mutex 没释放！
    ...
}
```

> → 理发师**拿着锁睡觉** → 所有顾客卡在 `P(mutex)` → **死锁** 💀

**正确顺序**：`P(customer)` 必须在 `P(mutex)` **之前**。

---

## 五、实战例题

### 题目 1：理发店（经典问题）

> 1 位理发师、1 把理发椅、n 把等候椅。没顾客理发师睡觉；满员顾客离开。

**= 模板 2**，将 `M` 换成 `n` 即可。

---

### 题目 2：银行（店满离开）

> 1 个服务窗口、10 个等候座。满员离开。取号机互斥使用。

**= 模板 2**，用 `cobegin` 语法重写：

```c
semaphore customer = 0;
semaphore server   = 0;
semaphore mutex    = 1;
int waiting        = 0;

cobegin {
    process 顾客_i {
        P(mutex);
        if (waiting < 10) {
            从取号机获取一个号码;
            waiting++;
            V(mutex);
            V(customer);
            P(server);
            等待叫号;
            获取服务;
        } else {
            V(mutex);
            离开;
        }
    }

    process 营业员 {
        while (TRUE) {
            P(customer);
            P(mutex);
            叫号;
            waiting--;
            V(mutex);
            V(server);
            为客户服务;
        }
    }
}
```

---

### 题目 3：银行（店满等待）

> 和题目 2 唯一区别：满员**不离开，等待空位**。

**= 模板 5**，把 `waiting` 变量换成 `empty` 信号量：

```c
semaphore customer = 0;
semaphore server   = 0;
semaphore empty    = 10;     // ← 信号量代替 waiting
semaphore mutex    = 1;

cobegin {
    process 顾客_i {
        P(empty);            // 没空位 → 阻塞等
        P(mutex);
        从取号机获取一个号码;
        V(mutex);
        V(customer);
        P(server);
        等待叫号;
        获取服务;
    }

    process 营业员 {
        while (TRUE) {
            P(customer);
            P(mutex);
            叫号;
            V(mutex);
            V(empty);        // 服务完释放一个空位
            V(server);
            为客户服务;
        }
    }
}
```

---

## 六、学习策略

1. **先死记模板 1**——所有变体的骨架
2. 做题时**先默写模板 1** → 按题意修补
3. 满员**离开** → 用 `waiting` 变量 + `if`（模板 2/3/4）
4. 满员**等待** → 用 `empty` 信号量 + `P(empty)`（模板 5）
5. **服务人员数量 N 是冗余条件**，解题时可忽略

---

## 七、与生产者-消费者问题的对照

| | 生产者-消费者 | 理发师问题 |
|:---|:---|:---|
| 缓冲区 | 产品队列 | 等候椅 |
| 生产者 | 生产者进程 | 顾客进程 |
| 消费者 | 消费者进程 | 服务人员进程 |
| 同步1 | `empty`（空位数） | `customer`（顾客资源） |
| 同步2 | `full`（产品数） | `server`（服务人员资源） |
| 互斥 | `mutex` | `mutex` |
| 满时行为 | 生产者阻塞等待 | 离开 or 等待 |

> 本质上理发师问题是生产者-消费者问题的**双向对称写法**。
