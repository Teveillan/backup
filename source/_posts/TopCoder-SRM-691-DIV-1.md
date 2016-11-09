title: "TopCoder SRM 691 Div1 "
date: 2016-05-31 19:42:12
categories: 红名之路
tags: [topcoder]
description: 
---

##Easy: Sunnygraphs
题意： 最初有n个点，每个点i与a[i]相连，另外又一个点P。
在原先的n个点中，选一个点的集合S，对于任意点 $~i \in S$，进行以下两种操作：
* 删除i与a[i]的那条边
* 添加i与P的边

<!--more-->
问有多少种点的集合S，使得0与1最后相连。

当图为有向图时，我们设：
* 0能到达的点集为A；
* 1能到达的点集为B；

分一下两种情况：
1. 当0 1不连通（无向图）的时候， 自然从A至少选一个点，从B中至少选一个点。
2. 当0 1连通（无向图）的时候。以下两种情况不符合要求：
	* 存在一个点$x \in S$，满足$x \in (A - B)  且 S \bigcap B = \emptyset $。也就是说有个0能到而1却不能到达的点x在集合S中 ，且B能到达的点不在S中。
	* 存在一个点$x \in S$，满足$x \in (B - A)  且 S \bigcap A = \emptyset $。

```c++
const int N = 55;
bool g[N][N], f[N][N];

class Sunnygraphs
{
        public:
        long long count(vector <int> a) {
            memset(g, 0, sizeof(g)); //有向图的连通性
            memset(f, 0, sizeof(f)); //无向图的连通性
            int n = a.size();
            REP(i, n) g[i][i] = 1;
            REP(i, n) g[i][a[i]] = 1, f[i][a[i]] = f[a[i]][i] = 1;
            REP(k, n) REP(i, n) REP(j, n) g[i][j] |= (g[i][k] & g[k][j]), f[i][j] |= (f[i][k] & f[k][j]);
            int num0 = 0, num1 = 0, num2 = 0;
            REP(i, n) {
                if(g[0][i] && g[1][i]) ++num2;
                else if(g[0][i]) ++num0;
                else if(g[1][i]) ++num1;
            }
            if(!f[0][1]) {
                int tmp = n - num0 - num1;
                return ((1ll<<num0) - 1) * ((1ll<<num1) - 1) * (1ll << tmp);
            }
            int tmp = n - num0 - num1 - num2;
            LL ret = 1ll << n;
            ret -= (((1ll << num0) - 1 )* (1ll << tmp));
            ret -= (((1ll << num1) - 1) * (1ll << tmp));
            return ret;
        }
}
```

## Medium: Money Manager
题意： 有n个项目，每做一个项目有经验值$a_i$以及基础奖金$b_i$。
每做完一个项目，设历史积攒的经验值总和为exp，则做完这个项目能得到的奖金为$b_i$ * exp。
另外，在第`n/2`个项目做完时，需要做一个项目，增加的经验值为X，但没有奖金。
问怎么安排项目顺序使得最后能获得的奖金总和最大。

我们先考虑在没有X的情况下的方案。
有一个性质： 当 $ a_i \* b_j > a_j \* b_i $时， 则i必定在j之前。且这个式子是有传递性的。

也就是说，当不考虑X的时候，用排序算法就能解决。如果我们选好了前n个需要做的项目有哪些，则相对顺序是确定了的。
当考虑X时，我们发现，影响因素仅仅是后n个项目的b之和。
因此，枚举后n个项目的b之和（假设为f）。
DP[l][r][sr], 表示前n个项目中以及选好了l个，后n个项目中已经选好了r个，且后n个项目的b之和为sr。

```c++
bool cmp(const pii &a, const pii &b) {
    return a.first * b.second < a.second * b.first;
}

class Moneymanager {
    int dp[30][30][300];
    int sum[55];
    void setmax(int &x, int y) {x = max(x, y);}
    public:
    int getbest(vector <int> a, vector <int> b, int X) {
        int n = a.size(), m = n / 2;
        vector<pii> vec;
        REP(i, n) vec.pb(pii(a[i], b[i]));
        sort(vec.begin(), vec.end(), cmp);

        REP(i, n) {
            sum[i] = vec[i].second;
            if(i) sum[i] += sum[i-1];
        }

        int ret = 0;
        FOR(f, 0, m * 10) {
            memset(dp, -1, sizeof(dp));
            dp[0][0][0] = f * X;

            FOR(l, 0, m) FOR(r, 0, m) {
                if(l == m && r == m) continue;
                int a = vec[l+r].first, b = vec[l+r].second;

                FOR(sr, 0, r * 10) {
                    int sl = sum[l + r] - sr - b;
                    if(sl < 0) continue;
                    setmax(dp[l+1][r][sr], dp[l][r][sr] + a * (sl + b) + a * f);
                    setmax(dp[l][r+1][sr+b], dp[l][r][sr] + a * (sr + b));
                }
            }
            setmax(ret, dp[m][m][f]);
        }
        return ret;
    }
}
```
