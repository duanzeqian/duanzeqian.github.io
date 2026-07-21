---
date: 2026-07-21
author: Zeqian Duan
---

# **[ACMOJ2928 导弹拦截](https://acm.sjtu.edu.cn/OnlineJudge/problem/2928) 题解**

本题可以分成两个子问题来解决。

**第一个子问题**是求出所有导弹的威力值 $w_j$。由平抛运动的特性，显然只有初始高度相同的导弹才可能发生碰撞。

固定某个高度 $H$，下落用时为 $t = \sqrt{\frac{2H}{g}}$。由于数据保证了每个导弹的初始位置不同，因此同一高度下每个导弹的初始位置不同。

考虑当两个导弹发生碰撞的时刻，显然无论这两个导弹是相遇还是追及，它们都有且仅有这一个时刻位置会重合，且在这一时刻前后两个导弹的相对位置是互换的。

因而，本质上，导弹的威力值就是它与别的导弹位置交换发生的次数。我们可以自然地联想到以前小作业中的一道题[ACMOJ1301 Bubbling Bubbles](https://acm.sjtu.edu.cn/OnlineJudge/problem/1301)。

所以第一个子问题可以用分治解决，上课时也分享过逆序对的BIT写法，两种解法均可。这里只给出BIT写法：
```
for (int i = 0; i < siz; ++i)
{
    d[g[id][i].idx].w += getSum(siz) - getSum(num[i] - 1); // g[id]为一批同时落地、会发生碰撞的导弹，不同id的导弹互不干涉
    Update(num[i], siz);
} 
for (int i = 1; i <= siz; ++i) c[i] = 0;
for (int i = siz - 1; ~i; --i)
{
    d[g[id][i].idx].w += getSum(num[i]);
    Update(num[i], siz);
}
```

注意题目要求“导弹落地时发生碰撞也会增加威力”，所以它求的并非严格逆序对，在算到 $num_i$ 时也需要将BIT中先前所存的 $num_i$ 的数据也加上。

由于 $0 \leq |x_j| \leq 10^9$，需要将坐标数据离散化。


**第二个子问题**中，每个问题询问 $T$ 时刻时 $l$ 与 $r$ 之间的被摧毁的城市数。维护一个数组 $a_n$ 记录每个城市的拦截能力，维护一个BIT记录被摧毁的城市。

先将所有询问按照 $T$ 的大小升序排序，再遍历每次询问，当到了某个 $T$ 时，一批新的导弹开始打击城市时，就更新对应城市的拦截能力 $a_i$，当 $a_i$ 由非负变为负数的时候，第 $i$ 座城市被摧毁，在BIT中将对应下标的权值加一。

对于每次询问，在BIT中查询对应下标区间的区间和即是被摧毁的城市个数。

在样例代码的实现中，以 $vis_n$ 标记一座城市是否被摧毁，在你的实现中也可以不使用。

```
for (int i = 1; i <= Q; ++i)
{
    while (id <= m && d[id].t <= q[i].T + eps) // epsilon为精度误差
    {
        ll cid = d[id].b; // 对应导弹的打击城市
        if (vis[cid]) // 已经被摧毁 
        {
            id++;
            continue;
        }
        a[cid] -= d[id].w; // 对应城市的拦截能力
        if (a[cid] < 0)
        {
            Update(cid, n);
            vis[cid] = true;
        }
        id++;
    }
    ans[q[i].idx] = getSum(q[i].r) - getSum(q[i].l - 1); 
}
```