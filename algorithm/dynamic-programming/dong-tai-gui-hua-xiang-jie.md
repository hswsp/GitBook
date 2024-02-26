# 动态规划详解

## Dynamic Programming代码框架

动态规划三要素：**重叠⼦问题**、**最优⼦结构**、**状态转移⽅程**。

动态规划的三个需要明确的点就是「状态」「选择」和「base case」，对应着回溯算法中走过的「路径」，当前的「选择列表」和「结束条件」。

某种程度上说，动态规划的**暴力求解阶段就是回溯算法**。只是有的问题具有重叠子问题性质，可以用 dp table 或者备忘录优化，将递归树大幅剪枝，这就变成了动态规划。

思维框架：**明确「状态」 -> 定义 dp 数组/函数的含义 -> 明确「选择」-> 明确 base case。**

解题方法: **状态表示 ->写出状态转移方程 ->确定边界 ->如果用递推，考虑子状态枚举的顺序**

### 记忆搜索格式：

```cpp
int dp[状态]
int dfs(状态)
{
    if(决策边界)
        return 决策边界答案；
    if(dp[状态表示]!=无效数值)
        return dp[状态表示]；
    for(当前状态的子状态)
        dfs(子状态)；
        更新dp[状态表示]；
    return dp[状态表示]；
}

int solve()
{
    memset(dp,无效数值，sizeof(dp));
    return dfs(原问题状态)；
}
```

### **dp 数组的迭代格式**&#x20;

```cpp
int solve(inputs,状态) {
    vector<int> dp(状态);
    // base case
    dp[决策边界答案] = ...;
    for(按合适的遍历方向){
       更新dp[状态表示]；
    }
    return dp[结果状态];
}
```

## 列题说明

#### Leetcode 64 minimum path sum

Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

Note: You can only move either down or right at any point in time.

> Example:
>
> Input: \[ \[1,3,1], \[1,5,1], \[4,2,1] ]

Output: 7 Explanation: Because the path 1→3→1→1→1 minimizes the sum.

### **记忆搜索格式**

```cpp
int dp[550][550]
vector<vector<int>> grid;

int dfs(int x, int y)
{
    if(x==0&& y==0) return grid[0][0];
    if(x<0 || y<0) return INT_MAX;
    if(dp[x][y]!=-1) return dp[x][y];

    dp[x][y] = min(dfs(x-1,y),dfs(x,y-1))+grid[x][y];

    return dp[x][y]；
}
```

### **递推表示**

```cpp
   int minpathsum(vector<vector<int>>& grid)
   {
       int n = grid.size(),m = grid[0].size();
       int dp[n][m] = 0;
       //边界
       dp[0][0] = grid[0][0];
       for(int i = 1;i<n;i++) dp[i][0] = dp[i-1][0]+grid[i][0];
       for(int j=0;j<m;j++) dp[0][j] = dp[0][j-1]+grid[0][j];
       //状态转移
       for(i=0;i<n;++i)
           for(j = 0;j<m;++j)
               dp[i][j] = min(dp[i-1][j],dp[i][j-1])+grid[i][j];
   }
```

#### **注：最大公约数**

```cpp
int gcd(a,b)
{
    if(b==0) return a;
    else return gcd(b,a%b);
}
```

## 最优子结构详解

「最优子结构」是某些问题的一种特定性质，并不是动态规划问题专有的。也就是说，很多问题其实都具有最优子结构，只是其中大部分不具有重叠子问题，所以我们不把它们归为动态规划系列问题而已。

我先举个很容易理解的例子：假设你们学校有 10 个班，你已经计算出了每个班的最高考试成绩。那么现在我要求你计算全校最高的成绩，你会不会算？当然会，而且你不用重新遍历全校学生的分数进行比较，而是只要在这 10 个最高成绩中取最大的就是全校的最高成绩。

我给你提出的这个问题就**符合最优子结构**：**可以从子问题的最优结果推出更大规模问题的最优结果**。让你算**每个班**的最优成绩就是子问题，你知道所有子问题的答案后，就可以借此推出**全校**学生的最优成绩这个规模更大的问题的答案。

你看，这么简单的问题都有最优子结构性质，只是因为显然没有重叠子问题，所以我们简单地求最值肯定用不出动态规划。

再举个例子：假设你们学校有 10 个班，你已知每个班的最大分数差（最高分和最低分的差值）。那么现在我让你计算全校学生中的最大分数差，你会不会算？可以想办法算，但是肯定不能通过已知的这 10 个班的最大分数差推到出来。因为这 10 个班的最大分数差不一定就包含全校学生的最大分数差，比如全校的最大分数差可能是 3 班的最高分和 6 班的最低分之差。

这次我给你提出的问题就**不符合最优子结构**，因为你没办通过每个班的最优值推出全校的最优值，没办法通过子问题的最优值推出规模更大的问题的最优值。前文 [动态规划详解](http://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484731\&idx=1\&sn=f1db6dee2c8e70c42240aead9fd224e6\&chksm=9bd7fb33aca07225bee0b23a911c30295e0b90f393af75eca377caa4598ffb203549e1768336\&scene=21#wechat\_redirect) 说过，**想满足最优子结，子问题之间必须互相独立。**全校的最大分数差可能出现在两个班之间，显然子问题不独立，所以这个问题本身不符合最优子结构。

**那么遇到这种最优子结构失效情况，怎么办？策略是：改造问题**。对于最大分数差这个问题，我们不是没办法利用已知的每个班的分数差吗，那我只能这样写一段暴力代码：

```cpp
int result = 0;
for (Student a : school) {
    for (Student b : school) {
        if (a is b) continue;
        result = max(result, |a.score - b.score|);
    }
}
return result;
```

改造问题，也就是把问题等价转化：最大分数差，不就等价于最高分数和最低分数的差么，那不就是要求最高和最低分数么，不就是我们讨论的第一个问题么，不就具有最优子结构了么？那现在改变思路，借助最优子结构解决最值问题，再回过头解决最大分数差问题，是不是就高效多了？

当然，上面这个例子太简单了，不过请读者回顾一下，我们做动态规划问题，是不是一直在求各种最值，本质跟我们举的例子没啥区别，无非需要处理一下重叠子问题。

前文 [动态规划：不同的定义产生不同的解法](http://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484469\&idx=1\&sn=e8d321c8ad62483874a997e9dd72da8f\&chksm=9bd7fa3daca0732b316aa0afa58e70357e1cb7ab1fe0855d06bc4a852abb1b434c01c7dd19d6\&scene=21#wechat\_redirect) 和 [经典动态规划：高楼扔鸡蛋（进阶篇）](http://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484690\&idx=1\&sn=eea075701a5d96dd5c6e3dc6a993cac5\&chksm=9bd7fb1aaca0720c58c9d9e02a8b9211a289bcea359633a95886d7808d2846898d489ce98078\&scene=21#wechat\_redirect) 就展示了如何改造问题，不同的最优子结构，可能导致不同的解法和效率。

再举个常见但也十分简单的例子，求一棵二叉树的最大值，不难吧（简单起见，假设节点中的值都是非负数）：

```cpp
int maxVal(TreeNode root) {
    if (root == null)
        return -1;
    int left = maxVal(root.left);
    int right = maxVal(root.right);
    return max(root.val, left, right);
}
```

你看这个问题也符合最优子结构，以`root`为根的树的最大值，可以通过两边子树（子问题）的最大值推导出来，结合刚才学校和班级的例子，很容易理解吧。

当然这也不是动态规划问题，旨在说明，最优子结构并不是动态规划独有的一种性质，能求最值的问题大部分都具有这个性质；**但反过来，最优子结构性质作为动态规划问题的必要条件，一定是让你求最值的**，以后碰到那种恶心人的**最值题**，思路往动态规划想就对了，这就是套路。

动态规划不就是从最简单的 base case 往后推导吗，可以想象成一个链式反应，不断以小博大。但只有符合最优子结构的问题，才有发生这种链式反应的性质。

**找最优子结构的过程，其实就是证明状态转移方程正确性的过程**，方程符合最优子结构就可以写暴力解了，写出暴力解就可以看出有没有重叠子问题了，有则优化，无则 OK。这也是套路，经常刷题的朋友应该能体会。

这里就不举那些正宗动态规划的例子了，读者可以翻翻历史文章，看看状态转移是如何遵循最优子结构的，这个话题就聊到这，下面再来看另外个动态规划迷惑行为。

## dp 数组的遍历方向

我相信读者做动态规划问题时，肯定会对`dp`数组的遍历顺序有些头疼。我们拿二维`dp`数组来举例，有时候我们是正向遍历：

```cpp
int[][] dp = new int[m][n];
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        // 计算 dp[i][j]
```

有时候我们反向遍历：

```cpp
for (int i = m - 1; i >= 0; i--)
    for (int j = n - 1; j >= 0; j--)
        // 计算 dp[i][j]
```

有时候可能会斜向遍历：

```cpp
// 斜着遍历数组,忽略对角线的 Base Case
for (int l = 1; l <= n-1; l++) {
    for (int i = 0; i <= n - 1 - l; i++) {
        int j = i+l;
        // 计算 dp[i][j]
    }
}
```

甚至更让人迷惑的是，有时候发现正向反向遍历都可以得到正确答案，比如我们在 [团灭 LeetCode 股票买卖问题](http://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484508\&idx=1\&sn=42cae6e7c5ccab1f156a83ea65b00b78\&chksm=9bd7fa54aca07342d12ae149dac3dfa76dc42bcdd55df2c71e78f92dedbbcbdb36dec56ac13b\&scene=21#wechat\_redirect) 中有的地方就正反皆可。

那么，如果仔细观察的话可以发现其中的原因的。你只要把住两点就行了：

**1、遍历的过程中，所需的状态必须是已经计算出来的**。

**2、遍历的终点必须是存储结果的那个位置**。

下面来具体解释上面两个原则是什么意思。

比如编辑距离这个经典的问题，详解见前文 [经典动态规划：编辑距离](http://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484484\&idx=1\&sn=74594297022c84952162a68b7f739133\&chksm=9bd7fa4caca0735a1364dd13901311ecd6ec4913c8db05a1ff6cae8f069627eebe8d651bbeb1\&scene=21#wechat\_redirect)，我们通过对`dp`数组的定义，确定了 base case 是`dp[..][0]`和`dp[0][..]`，最终答案是`dp[m][n]`；而且我们通过状态转移方程知道`dp[i][j]`需要从`dp[i-1][j]`,`dp[i][j-1]`,`dp[i-1][j-1]`转移而来，如下图：

![](<../.gitbook/assets/image (25).png>)

那么，参考刚才说的两条原则，你该怎么遍历`dp`数组？肯定是正向遍历：

```cpp
for (int i = 1; i < m; i++)
    for (int j = 1; j < n; j++)
        // 通过 dp[i-1][j], dp[i][j - 1], dp[i-1][j-1]
        // 计算 dp[i][j]
```

**因为，这样每一步迭代的左边、上边、左上边的位置都是 base case 或者之前计算过的，而且最终结束在我们想要的答案`dp[m][n]`。**

再举一例，回文子序列问题，详见前文 [子序列解题模板：最长回文子序列](http://mp.weixin.qq.com/s?\_\_biz=MzAxODQxMDM0Mw==\&mid=2247484666\&idx=1\&sn=e3305be9513eaa16f7f1568c0892a468\&chksm=9bd7faf2aca073e4f08332a706b7c10af877fee3993aac4dae86d05783d3d0df31844287104e\&scene=21#wechat\_redirect)，我们通过过对`dp`数组的定义，确定了 base case 处在中间的对角线，`dp[i][j]`需要从`dp[i+1][j]`,`dp[i][j-1]`,`dp[i+1][j-1]`转移而来，想要求的最终答案是`dp[0][n-1]`，如下图：

![](<../.gitbook/assets/image (14).png>)

这种情况根据刚才的两个原则，就可以有两种正确的遍历方式：

![](<../.gitbook/assets/image (55).png>)

**要么从左至右斜着遍历，要么从下向上从左到右遍历，这样才能保证每次`dp[i][j]`的左边、下边、左下边已经计算完毕，最终得到正确结果。**

现在，你应该理解了这两个原则，主要就是看 base case 和最终结果的存储位置，保证遍历过程中使用的数据都是计算完毕的就行，有时候确实存在多种方法可以得到正确答案，可根据个人口味自行选择。

## 状态压缩

能够使用状态压缩技巧的动态规划都是二维`dp`问题，**你看它的状态转移方程，如果计算状态`dp[i][j]`需要的都是`dp[i][j]`相邻的状态，那么就可以使用状态压缩技巧**，将二维的`dp`数组转化成一维，将空间复杂度从 O(N^2) 降低到 O(N)。

什么叫「和`dp[i][j]`相邻的状态」呢，比如 「最长回文子序列」 最终的代码如下：

```cpp
int longestPalindromeSubseq(string s) {
    int n = s.size();
    // dp 数组全部初始化为 0
    vector<vector<int>> dp(n, vector<int>(n, 0));
    // base case
    for (int i = 0; i < n; i++)
        dp[i][i] = 1;
    // 反着遍历保证正确的状态转移
    for (int i = n - 2; i >= 0; i--) {
        for (int j = i + 1; j < n; j++) {
            // 状态转移方程
            if (s[i] == s[j])
                dp[i][j] = dp[i + 1][j - 1] + 2;
            else
                dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
        }
    }
    // 整个 s 的最长回文子串长度
    return dp[0][n - 1];
}
```

PS：我们本文不探讨如何推状态转移方程，只探讨对二维 DP 问题进行状态压缩的技巧。**技巧都是通用的，所以如果你没看过前文，不明白这段代码的逻辑也无妨，完全不会阻碍你学会状态压缩。**

你看我们对`dp[i][j]`的更新，其实只依赖于`dp[i+1][j-1], dp[i][j-1], dp[i+1][j]`这三个状态：

![](<../.gitbook/assets/image (12).png>)

这就叫和`dp[i][j]`相邻，反正你计算`dp[i][j]`只需要这三个相邻状态，其实根本不需要那么大一个二维的 dp table 对不对？

**状态压缩的核心思路就是，将二维数组「投影」到一维数组**：

![](<../.gitbook/assets/image (33).png>)

思路很直观，但是也有一个明显的问题，图中`dp[i][j-1]`和`dp[i+1][j-1]`这两个状态处在同一列，而一维数组中只能容下一个，那么当我计算`dp[i][j]`时，他俩必然有一个会被另一个覆盖掉，怎么办？

这就是状态压缩的难点，下面就来分析解决这个问题，还是拿「最长回文子序列」问题举例，它的状态转移方程主要逻辑就是如下这段代码：

```cpp
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        // 状态转移方程
        if (s[i] == s[j])
            dp[i][j] = dp[i + 1][j - 1] + 2;
        else
            dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
    }
}
```

想把二维`dp`数组压缩成一维，一般来说是把第一个维度，也就是`i`这个维度去掉，只剩下`j`这个维度。**压缩后的一维`dp`数组就是之前二维`dp`数组的`dp[i][..]`那一行**。

我们先将上述代码进行改造，直接无脑去掉`i`这个维度，把`dp`数组变成一维：

```cpp
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        // 在这里，一维 dp 数组中的数是什么？
        if (s[i] == s[j])
            dp[j] = dp[j - 1] + 2;
        else
            dp[j] = max(dp[j], dp[j - 1]);
    }
}
```

上述代码的一维`dp`数组只能表示二维`dp`数组的一行`dp[i][..]`，那我怎么才能得到`dp[i+1][j-1], dp[i][j-1], dp[i+1][j]`这几个必要的的值，进行状态转移呢？

在代码中注释的位置，将要进行状态转移，更新`dp[j]`，那么我们要来思考两个问题：

1、在对`dp[j]`赋新值之前，`dp[j]`对应着二维`dp`数组中的什么位置？

2、`dp[j-1]`对应着二维`dp`数组中的什么位置？

**对于问题 1，在对`dp[j]`赋新值之前，`dp[j]`的值就是外层 for 循环上一次迭代算出来的值，也就是对应二维`dp`数组中`dp[i+1][j]`的位置**。（二维投影到一维，唯一投影）

**对于问题 2，`dp[j-1]`的值就是内层 for 循环上一次迭代算出来的值，也就是对应二维`dp`数组中`dp[i][j-1]`的位置**。（二维投影到一维，多值投影，之前的都被覆盖掉了，只剩下最后一个`dp[i][j-1]`)

那么问题已经解决了一大半了，只剩下二维`dp`数组中的`dp[i+1][j-1]`这个状态我们不能直接从一维`dp`数组中得到：

```cpp
for (int i = n - 2; i >= 0; i--) {
    for (int j = i + 1; j < n; j++) {
        if (s[i] == s[j])
            // dp[i][j] = dp[i+1][j-1] + 2;
            dp[j] = ?? + 2;
        else
            // dp[i][j] = max(dp[i+1][j], dp[i][j-1]);
            dp[j] = max(dp[j], dp[j - 1]);
    }
}
```

因为 for 循环遍历`i`和`j`的顺序为从左向右，从下向上，所以可以发现，在更新一维`dp`数组的时候，`dp[i+1][j-1]`会被`dp[i][j-1]`覆盖掉，图中标出了这四个位置被遍历到的次序：

![](<../.gitbook/assets/image (86).png>)

**那么如果我们想得到`dp[i+1][j-1]`，就必须在它被覆盖之前用一个临时变量`temp`把它存起来，并把这个变量的值保留到计算`dp[i][j]`的时候**。为了达到这个目的，结合上图，我们可以这样写代码：

```cpp
for (int i = n - 2; i >= 0; i--) {
    // 存储 dp[i+1][j-1] 的变量
    int pre = 0;
    for (int j = i + 1; j < n; j++) {
        int temp = dp[j];
        if (s[i] == s[j])
            // dp[i][j] = dp[i+1][j-1] + 2;
            dp[j] = pre + 2;
        else
            dp[j] = max(dp[j], dp[j - 1]);        // 到下一轮循环，pre 就是 dp[i+1][j-1] 了
        pre = temp;
    }
}
```

别小看这段代码，这是一维`dp`最精妙的地方，会者不难，难者不会。为了清晰起见，我用具体的数值来拆解这个逻辑：

假设现在`i = 5, j = 7`且`s[5] == s[7]`，那么现在会进入下面这个逻辑对吧：

```cpp
if (s[5] == s[7])
    // dp[5][7] = dp[i+1][j-1] + 2;
    dp[7] = pre + 2;
```

我问你这个`pre`变量是什么？是内层 for 循环上一次迭代的`temp`值。

那我再问你内层 for 循环上一次迭代的`temp`值是什么？是`dp[j-1]`也就是`dp[6]`，但这是外层 for 循环上一次迭代对应的`dp[6]`，也就是二维`dp`数组中的`dp[i+1][6] = dp[6][6]`。

也就是说，`pre`变量就是`dp[i+1][j-1] = dp[6][6]`，也就是我们想要的结果。

那么现在我们成功对状态转移方程进行了降维打击，算是最硬的的骨头啃掉了，但注意到我们还有 base case 要处理呀：

```cpp
// 二维 dp 数组全部初始化为 0
vector<vector<int>> dp(n, vector<int>(n, 0));
// base case
for (int i = 0; i < n; i++)
    dp[i][i] = 1;
```

如何把 base case 也打成一维呢？**很简单，记住，状态压缩就是投影，**我们把 base case 投影到一维看看：

![](<../.gitbook/assets/image (40).png>)

二维`dp`数组中的 base case 全都落入了一维`dp`数组，不存在冲突和覆盖，所以说我们直接这样写代码就行了：

```cpp
// 一维 dp 数组全部初始化为 1
vector<int> dp(n, 1);
```

至此，我们把 base case 和状态转移方程都进行了降维，实际上已经写出完整代码了：

```cpp
int longestPalindromeSubseq(string s) {
    int n = s.size();
    // base case：一维 dp 数组全部初始化为 1
    vector<int> dp(n, 1);

    for (int i = n - 2; i >= 0; i--) {
        int pre = 0;
        for (int j = i + 1; j < n; j++) {
            int temp = dp[j];
            // 状态转移方程
            if (s[i] == s[j])
                dp[j] = pre + 2;
            else
                dp[j] = max(dp[j], dp[j - 1]);
            pre = temp;
        }
    }
    return dp[n - 1];
}
```

使用状态压缩技巧对二维`dp`数组进行降维打击之后，解法代码的可读性变得非常差了，如果直接看这种解法，任何人都是一脸懵逼的。

算法的优化就是这么一个过程，先写出可读性很好的暴力递归算法，然后尝试运用动态规划技巧优化重叠子问题，最后尝试用状态压缩技巧优化空间复杂度。
