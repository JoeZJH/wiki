---
title: "谱图理论-Spectral Graph Theory"
layout: page
date: 2019-11-05
---
[TOC]

# 关于
- 课程 Spectral Graph Theory  的学习笔记
- <http://www.cs.yale.edu/homes/spielman/561/>
- python 图代码库 <https://networkx.github.io/> <https://networkx.github.io/documentation/stable/>

# 导言
- 图的邻接矩阵 $(A_G(u, v))$
- 图的操作：
    - diffusion operator，扩散操作：每一个时刻，某个顶点上的东西，均匀扩散到邻居顶点，不保留任何物质
        - $(D_G)$表示顶点的度矩阵，那么考虑归一化（这是为了得到扩散概率）邻接矩阵$(W_G = D_G^{-1} A_G)$ 。它的每一行$(W_G(u, \cdot))$代表从顶点u通过一次扩散操作转移到其他顶点的物质的比例。
            - 另一种解释：W可以看做一个概率转移矩阵，从u转移到v的概率
        - 用向量p表示初始每个顶点物质的数量，那么进过一次扩散操作后，每个顶点物质的数量为 $(p W_G)$
            - 概率解释：一次扩散操作后，每个定点上的物质的数量是所有定点的量通过一次转移，到达该顶点的量的总和
    - 拉普拉斯矩阵：$(L_G = D_G - A_G)$
        - 半正定矩阵 $(x^T L_G x  = \sum_{(u,v) \in E} (x(u) - x(v))^2)$
        - 二次型的意义是图上数量分布的平滑性
- 谱论 
    - 实对称矩阵，有n个特征值和对应的特征向量。线性代数的基础知识，不多介绍
    - 瑞利商 $(\frac{x^T M x}{x^T x})$
- 拉普拉斯矩阵的特征向量，假设特征值从小到大排序 $(\lambda_1 \le \lambda_2 \le \cdot \lambda_n)$
    - L的特征值大于等于0
    - 0 是特征值，对应的特征向量是全1向量
    - 特征值0的代数重数代表图可以划分为互不联通的子图的数目
    - $(\lambda_2 > 0)$ 当且仅当图是联通的。
        - 个人理解，一个互相联通的图可以用一个拉普拉斯矩阵表示出来，且存在一个非0向量使得这个Lx=0
        - 两个互不联通的图看做一个图的时候，由于两个图节点间没有任何连接，因此对应的总拉普拉斯矩阵应该是
        $$
        L = \begin{bmatrix}
        L_1 & O \\\\
        O & L_2
        \end{bmatrix}
        $$
        - 因此，显然可以构造两个不同的向量$([v_1, O], [O, v_2])$ 使得Lv=0，其中v1和v2分别是子矩阵的0空间中的任意向量。
    

# 拉普拉斯矩阵
- 全1向量是特征值0对应的特征向量
- $(\lambda_2)$ 对应的特征向量是满足下列条件的向量

$$
\min_x x^T Lx \\\\
s.t. ||x||^2 = 1 \\\\
1^T x = 0
$$

- $(\lambda_3)$ 对应的特征向量是满足于跟上述x正交且满足上述条件的解y
- $(\lambda_2)$ 跟问题「通过切割最少的边，将图切分成两个子图」有密切关系


## Isoperimetry
- 顶点集合的子集S的边界(boundary)定义为S与剩下定点的所有相连的边的集合


$$
\partial S = \\{(u,v) \in E| u \in S, v\notin S \\}
$$

- isoperimetric ratio 定义为S的边界集合大小与S顶点数目之比
$$
\theta(S) = \frac{|\partial S|}{|S|}
$$

- isoperimetric number 定义为上述比值的最小值，要求S的大小不超过所有顶点数的一半
$$
\theta_G = \min_{|S| \le n/2} \theta(S)
$$

- 定理(下界)：对每一个子集 $(S \subset V)$， 
$$
\theta(S) \ge \lambda_2 (1-s)
$$
s=|S|/|V|是顶点数目比率，因为S的数目不超过V的一半，所以$(s \le 1/2)$，所以
$$
\theta_G \ge \lambda_2 /2
$$
 
> 定理的证明，关键是理解 $(x^T L x)$ 的几何意义。取一个特殊的向量x，x是子集S的示性函数，即xi代表第i个定点是否在S中，如果在则xi=1，否则xi=0。
> 那么 $( (Lx)_i , i \in S)$ 的意义则为第i个顶点与S外的其他定点的边的数目，把S中的所有i求和即得到边界$(\partial S)$ 的大小了。
> 所以 $(\theta(S) = \frac{x^T L x}{x^T x})$
> 显然上式是不小于$(\lambda_2)$ 的，但是还有更准确的下界。
> 将x按照两个子空间分解（0特征值的空间，即全1向量空间；正交补空间，即 $(x^T 1 = 0)$）
> $(x = y + s)$，$(y^T 1 = 0)$，s则为每一维都是s的向量，且$(s_i=|S|/|V| )$
> 所以继续推导有（注意s是L的特征向量，Ls=0）
> $$
> \theta(S) = \frac{x^T L x}{x^T x} \\\\
>           = \frac{y^T L y}{x^T x}   \ge \lambda_2 \frac{y^T y}{x^T x} \\\\
>           = \lambda_2 \frac{y^T y}{y^T y + s^T s}  =  \lambda_2 (1-s)
> $$

- 定理的意义：这个定理说明了$(\lambda_2)$跟L的联通性的关系，$(\lambda_2)$越大，说明L联通性越好（任意子集与其补集间的边越多）

### 推导过程中为什么不能直接从$(\frac{x^T L x}{x^T x})$得到下界$(\lambda_2)$
- 因为这里的x不一定全部在全1向量的补空间中，所以只能缩放到0，而不能缩放到$(\lambda_2)$

## 举例
- 完全图Kn，任意两个顶点都是互相连接的
- 星状图Sn，所有顶点跟且只跟第1个顶点相连
- Kn的拉普拉斯矩阵特征值：0，代数重数为1；n，代数重数为n-1。$(L_{K_n} = n I - 1^T 1)$
    - 所以，任意跟全1向量正交的向量$(\phi)$，都有$(L_{K_n}\phi = n \phi )$。即都是特征值n的特征向量。这样的特征向量空间维度当然是n-1，对应代数重数等于n-1
- 如果定点v和w都只跟同一个节点z相连，那么$(\delta_v - \delta_w)$ 是L的特征向量，对应的特征值为1；
- 同时，上述条件下，其他特征值对应的特征向量$(\phi)$必定与这个特征向量正交，所以$(\phi(v) = \phi(w))$
- Sn的拉普拉斯矩阵特征值：0，代数重数为1，特征向量为全1向量；1，代数重数为n-2，特征向量分别为$(\delta_i - \delta_{i+1}, i=1,2,...,n-2)$；n，代数重数为1。（因为Sn的迹为2n-2，所以剩下的特征值必定为n）
- Sn的特征值为n的特征向量求解：设为$(\phi)$，因为$(\phi(2)=\phi(3)=...\phi(n))，所以只有两个独立变量。假设$(\phi = [a, b, b, ..., b])$，因为跟全1向量正交，所以 a+(n-1)b=0，所以得到特征向量为 [n-1, -1, -1, ..., -1]
- 立方体：Let G = (V,E) and H = (W,F) be graphs. Then G×H is the graph with vertex set V ×W and edge set

$$
((v,w), (v', w)), (v,v') \in E \\\\
((v,w), (v, w')), (w,w') \in F
$$

- 重要例子：G是只有两个顶点{0,1}和一条边的图,特征值为0和2。立方体$(H_n = H_{n-1} X G)$。如果用比特来表示立方体的每个定点，可以发现，如果两个顶点的比特只相差一位，那么他们之间就有一条边。例如n=2的时候，是一个正方向，顶点依次为00,01,11,10
- 立方体的特征值与特征向量，如果G和H的特征值分别为$(\lambda_i)$ 和 $(\mu_i)$


# 邻接矩阵和图染色
- 邻接矩阵作用到一个向量上，相当于做了一次扩散操作：即每个节点对应向量的一个元素，一次扩散操作将邻居节点按照权重聚集到中间节点
$$
(Ax)(u) = \sum_{(u,v)\in E} w_{u,v} x(v)
$$
- 假设A的特征值从大到小排列 $(\mu_1 \ge \mu_2 \ge ... \ge \mu_n)$
- 对于正规图（顶点的度都是d一样的）：因为$(L = I d - A)$，所以 $(\lambda_i = d - \mu_i)$
- 对于正规图：最大特征值是$(\mu_1 = d)$，对应全1向量
- 对于非正规图：$(d_{ave} \le \mu_1 \le d_{max})$
- 对mu1的下界进一步强化，设S是G的任意子图，有 $(d_{ave}(S) \le \mu_1)$
- 定理：实对称阵A的子对称矩阵（行下标和列下标完全一致的子矩阵）A(S)，其最大最小特征值被A的最大最小特征值夹着
$$
\lambda_{max}(A) \ge \lambda_{max}(A(S)) \ge \lambda_{min}(A(S)) \ge \lambda_{min}(A)
$$
- 如果G是联通的，那么$(\mu_1 = d_{max})$，此时叫$(d_{max})$-regular

- Perron-Frobenius, Symmetric Case：如果G是联通图，A是邻结矩阵，$(\mu_1 \ge \mu_2 \ge ... \ge \mu_n)$ 是特征值，有
    a. $(\mu_1 \ge -\mu_n)$
    b. $(\mu_1 > \mu_2)$
    c. $(\mu_1)$的存在特征向量$(\phi_1 \ge 0)$
- $(\mu_1 = -\mu_n)$ 当且仅当G是二分图

- 染色问题：保证相邻两个顶点的颜色不同，最少要$(\chi(G))$种不同的颜色。
- 将顶点编号，从小到大依次给顶点染色，保证顶点颜色跟小号顶点颜色不同即可。假设
$$
|\\{ v: v<u, (u, v) \in E \\}| \le k
$$
那么，最少可以用k+1中颜色对图进行染色。容易验证 $(k \le \lfloor \mu_1 \rfloor)$

- Hoffman 界
$$
\chi(G) \ge \frac{\mu_1 -\mu_n}{- \mu_n}
$$

# 特征值的界
- (Courant-Fischer Theorem). Let L be a symmetric matrix with eigenvalues λ1 ≤ λ2 ≤···≤λn. Then,
$$
\lambda_k = \min_{S \subset R^n, dim(S)=k} \max_{x \in S} \frac{x^T Ax}{x^T x} \\\\
 \max_{T \subset R^n, dim(T)=n - k + 1} \min_{x \in T} \frac{x^T Ax}{x^T x}
$$
- 这个定理是说，第k个特征值是前k个特征向量张成的子空间中所能取得的最大瑞丽商，是后n-k+1个特征向量张成的子空间中所得取得的最小瑞丽商
- $(\lambda_2)$ 是除了全1向量外的最小瑞利商。所以每一个跟全1向量正交的向量v的瑞利商给出一个$(\lambda_2)$的上界
- 令$(x_i = (n+1) - 2i)$，那么x跟1正交，所以导出一个上界$(\lambda_2 \le \frac{12}{n(n+1)})$
- 从拉普拉斯矩阵的二次型来看，增加边和增加边的权重，会增加二次型的值，所以有H>=G，H是在G增加边或者增加权重后构成的图

## 图的近似
- 图的近似，G的c-近似图H，如果满足 
$$
c H \ge G \ge H /c
$$

- 定理：对任意$(\epsilon > 0)$，存在 d >0，对充分大的n，有 d-regular 图Gn 是Kn的$(1+\epsilon)$近似
- 即如果n非常大的时候，可以用很少的边的图来近似很多的边的图！！（参考应用：NetSMF）
- 例1： $((n-1)P_n \ge G_{1,n})$，Pn表示n个顶点的路径图，即顶点依次相连的图；G1n表示只有第1，n个顶点之间相连的图
- 路径不等式，一般地，有$((j - i)P_{i,j} \ge G_{i,j})$
- 利用$(\lambda_2(K_n) = n)$可以推导出$(\lambda_2(P_n) \ge \frac{6}{(n+1)(n-1)} )$
- 如果图G和H有$(G \ge c H)$，那么$(\lambda_k(G) \ge c \lambda_k(H))$

## 完全二叉树
- 深度d的完全二叉树Td，节点数目为$(n=2^{d+1} -1)$，边集合为(i, 2i), (i, 2i+1)
- lambda2的上界，构造节点向量x，让根节点为0，左子树上的节点全为1，右子树上的节点券为-1.那么x跟1正交，且有
$$
\lambda_2(T_d) \le \frac{2}{n-1}
$$
- 利用完全图，可以得到另一个上界
$$
\lambda_2(T_d) \le \frac{1}{(n-1)\log_2 n}
$$


# 电导率与归一化拉普拉斯矩阵

## 电导率
- d(S)代表S中所有顶点的度之和；d(V)等于V中边数目的2倍
- 定义S的电导率
$$
\phi(S) = \frac{\partial S}{min(d(S), d(V-S))}
$$
- 图G的电导率
$$
\phi_G = \min_{S \subset V} \phi(S)
$$

## 归一化拉普拉斯矩阵
- 拉普拉斯矩阵归一化：$(N = D^)$
- 特征值$(0 = v_1 \le v_2 \le ... \le v_n)$
- 0特征值对应的特征向量是$(d^{1/2})$，d向量的每个元素是对应顶点的度
- 定理：$(v_2/2 \le \phi_G)$
- cheeger不等式 $(\phi_G \le \sqrt{2v_2})$

## Cheeger不等式


# Spring and Resistor Networks
- 想想图的边是一个个理想的弹簧，向量x代表每个定点的位置，x(u) - x(v) 代表u和v相对便宜，从而产生弹力 x(v) - x(u)，这里假设弹性系数是1（对应无权的边，对于有权图就是权重）
- 由受力平衡条件可得
$$
x(u) = \frac{1}{d(u)}\sum_{u: (u,v) \in E} x(v)
$$
- 对于加权图则为
$$
x(u) = \frac{1}{d(u)}\sum_{u: (u,v) \in E} w_{u,v} x(v)
$$
- 上式可以重写为拉普拉斯矩阵线性方程
$$
L x = 0
$$
- 设每个点有一个电压v和一个流出的电流i_ext，那么有 i_ext = Lv，边的权重是电导

- 顶点ab间的等效电阻可以通过设置$(i_{ext}=\delta_a - \delta_b)$得到
$$
R_{ab} = u_{ab} = (\delta_a - \delta_b)^T L^+ (\delta_a - \delta_b)
$$


# Random Walks on Graphs
## 转移概率
- $(p_t \in R^n)$ 定义在时刻t，各顶点在random walk下的概率分布，概率转移方程
$$
p_{t+1}(u) = \sum_{v: (u,v)\in E} \frac{w(u,v)}{d(v)}p_t(v)
$$
- lazy random walk, 以一定的概率(1/2)待在原地不动
$$
p_{t+1}(u) = \frac{1}{2} p_t(u) + \frac{1}{2} \sum_{v: (u,v)\in E} \frac{w(u,v)}{d(v)}p_t(v)
$$
## 扩散运动
- 气体/液体的扩散可以用上述两个方程来建模
- 扩散方程的矩阵形式
$$
p_{t+1} = 1/2(I + AD^{-1})p_t
$$
- lazy walk matrix $(W_G = 1/2(I + AD^{-1}) = I - \frac{1}{2} D^{-1/2}ND^{-1/2})$, N是归一化拉普拉斯矩阵
- 特征向量是$(D^{1/2}\phi_i)$, $(\phi_i)$是N的特征向量；特征值是$(1-v_i/2)$
- 稳态分布：顶点的概率正比于顶点的度
- 收敛率：初始值为$(e_a)$，那么对任意b，t
$$
|p_t(b) - \pi(b)| \le \frac{d(b)}{d(a)} w_2^t
$$
- w2是W的第2大特征值（第1大特征值为1）

## 举例
- 定理，拉普拉斯矩阵L，特征值$(\lambda_1 \le ... \le \lambda_n)$和归一化拉普拉斯矩阵N，特征值$(v_1 \le ... \le v_n)$，满足
$$
\frac{\lambda_i}{d_{min}} \ge v_i \ge \frac{\lambda_i}{d_{max}}
$$
- Path：度最大为2，最小为1，所以N的特征值近似为L的特征值，$(c/n^2)$
# FAQ

## 为什么特征值0的代数重数代表图可以划分为互不联通的子图的数目

两个互不联通的图看做一个图的时候，由于两个图节点间没有任何连接，因此对应的总拉普拉斯矩阵应该是
$$
L = \begin{bmatrix}
L_1 & O \\\\
O & L_2
\end{bmatrix}
$$

因此，显然可以构造两个不同的向量$([v_1, O], [O, v_2])$ 使得Lv=0，其中v1和v2分别是子矩阵的0空间中的任意向量。

显然，如果可以划分成k个子图，那么0特征值的代数重数就是k。



## 对拉普拉斯矩阵做线性变换有什么特殊含义吗


## 拉普拉斯矩阵的特征值和特征向量有什么意义


## 为什么联通图就是d-max regular？


## 如何理解Perron-Frobenius
- 注意A的元素都是非负的
- A相当于一次扩散操作
- 如果特征向量1中的某个元素为0，那么说明他的邻居里面有一些是负数，但是特征向量1是放大倍数最大的向量，所以如果把负数改成正数放大倍数更大，所以矛盾。
- mu1的放大倍数是最大的，不考虑方向也是


## 既然稳态分布是顶点的度，那为什么PageRank算法还有做特征向量分解？直接统计度不就行了？
