# 动态规划之正则表达

给你一个字符串 s 和一个字符规律 p，请你来实现一个支持 '.' 和 '\*' 的正则表达式匹配。

* '.' 匹配任意单个字符
* &#x20;'\*' 匹配零个或多个前面的那一个元素&#x20;

所谓匹配，是要涵盖 整个 字符串 s的，而不是部分字符串。

示例 1：

> 输入：s = "aa" p = "a"&#x20;
>
> 输出：false&#x20;
>
> 解释："a" 无法匹配 "aa" 整个字符串。



示例 2:

> 输入：s = "aa" p = "a\*"&#x20;
>
> 输出：true&#x20;
>
> 解释：因为'\*' 代表可以匹配零个或多个前面的那一个元素, 在这里前面的元素就是 'a'。因此，字符串 "aa" 可被视为 'a' 重复了一次。&#x20;

示例 3：

> 输入：s = "ab" p = ".\*"&#x20;
>
> 输出：true&#x20;
>
> 解释："_.\*_ " 表示可匹配零个或多个（'\*'）任意字符（'.'）。&#x20;

示例 4：

> 输入：s = "aab" p = "c\*a\*b"&#x20;
>
> 输出：true&#x20;
>
> 解释：因为 '\*' 表示零个或多个，这里 'c' 为 0 个, 'a' 被重复一次。因此可以匹配字符串 "aab"。

示例 5：

> 输入：s = "mississippi" p = "mis\*is\*p\*."&#x20;
>
> 输出：false

提示：

* 0 <= s.length <= 20&#x20;
* 0 <= p.length <= 30&#x20;
* s 可能为空，且只包含从 a-z 的小写字母。&#x20;
* p 可能为空，且只包含从 a-z 的小写字母，以及字符 . 和 _。_&#x20;
* 保证每次出现字符`*`时，前面都匹配到有效的字符

> Leetcode 10

## ⼀、不考虑通配符情况

第⼀步，我们暂时不管正则符号，如果是两个普通的字符串进⾏⽐较，如何 进⾏匹配？我想这个算法应该谁都会写：

```cpp
 bool isMatch(string text, string pattern) { 
        if (text.size() != pattern.size()) 
        return false; 
        for (int j = 0; j < pattern.size(); j++) { 
            if (pattern[j] != text[j]) 
            return false; 
        }
        return true; 
    }
```

稍微改造⼀下上⾯的代码，略微复杂了⼀点，但意思还是⼀样的:

```cpp
bool isMatch(string text, string pattern) { 
       int i = 0; // text 的索引位置 
       int j = 0; // pattern 的索引位置 
       while (j < pattern.size()) { 
           if (i >= text.size()) 
           return false; 
           if (pattern[j++] != text[i++]) 
           return false; 
        }// 相等则说明完成匹配 
        return j == text.size(); 
    }
}
```

如上改写，是为了将这个算法改造成递归算法（伪码）：

```python
def isMatch(text, pattern) -> bool: 
    if pattern is empty: return (text is empty?) 
    first_match = (text not empty) and pattern[0] == text[0] 
    return first_match and isMatch(text[1:], pattern[1:])
```



## ⼆、处理点号「.」通配符

点号可以匹配任意⼀个字符，稍加改造即可：

```python
def isMatch(text, pattern) -> bool: 
    if not pattern: return not text 
    first_match = bool(text) and pattern[0] in {text[0], '.'} 
    return first_match and isMatch(text[1:], pattern[1:])
```

## 三、处理「\*」通配符

星号通配符可以让前⼀个字符重复任意次数，包括零次。那到底是重复⼏次 呢？这似乎有点困难，不过不要着急，我们起码可以把框架的搭建再进⼀ 步：

```python
def isMatch(text, pattern) -> bool: 
    if not pattern: return not text 
    first_match = bool(text) and pattern[0] in {text[0], '.'} 
    if len(pattern) >= 2 and pattern[1] == '*': 
        # 发现 '*' 通配符 
    else:
        return first_match and isMatch(text[1:], pattern[1:])
```

星号前⾯的那个字符到底要重复⼏次呢？这需要计算机暴⼒穷举来算，假设重复 N 次吧。

前⽂多次强调过，**写递归的技巧是管好当下**，**之后的事抛给递归**。_**具体到这⾥，不管 N 是多少，当前的选择只有两个：匹配 0 次、匹配1次**_。所以可以这样处理：

```python
if len(pattern) >= 2 and pattern[1] == '*': 
    return isMatch(text, pattern[2:]) or \ 
        first_match and isMatch(text[1:], pattern) 
# 解释：如果发现有字符和 '*' 结合， 
# 或者匹配该字符 0 次，然后跳过该字符和 '*' 
# 或者当 pattern[0] 和 text[0] 匹配后，移动 text
```

可以看到，我们是通过保留 pattern 中的「\*」，同时向后推移 text，来实现将字符重复匹配多次的功能。举个简单的例⼦就能理解这个逻辑了。假 设 `pattern = a* , text = aa`，画个图看看匹配过程：

![](<../.gitbook/assets/Screen Shot 2021-05-27 at 1.33.32 PM.png>)

⾄此，正则表达式算法就基本完成了，

## 四、动态规划

将上述结果合并一下，就得到我们的暴力递归算法：

```python
# 暴⼒递归 
def isMatch(text, pattern) -> bool: 
    if not pattern: return not text 
    first = bool(text) and pattern[0] in {text[0], '.'} 
    
    if len(pattern) >= 2 and pattern[1] == '*': 
        return isMatch(text, pattern[2:]) or \ 
        first and isMatch(text[1:], pattern) 
    else:
        return first and isMatch(text[1:], pattern[1:])
```

我选择使⽤「备忘录」递归的⽅法来降低复杂度。有了暴⼒解法，优化的过程及其简单，就是使⽤两个变量 i, j 记录当前匹配到的位置，从⽽避免使⽤⼦字符串切⽚，并且将 i, j 存⼊备忘录，避免重复计算即可。

优化解法⽆⾮就是把暴⼒解法「翻译」了⼀遍，加了个 memo 作为备忘录，仅此⽽已。

```python
# 带备忘录的递归 
def isMatch(text, pattern) -> bool: 
    memo = dict() # 备忘录 
    def dp(i, j): 
        if (i, j) in memo: return memo[(i, j)] 
        
        if j == len(pattern): return i == len(text) 
        
        first = i < len(text) and pattern[j] in {text[i], '.'} 
        
        if j <= len(pattern) - 2 and pattern[j + 1] == '*': 
            ans = dp(i, j + 2) or first and dp(i + 1, j) 
        else:
            ans = first and dp(i + 1, j + 1) 
        memo[(i, j)] = ans 
        return ans 
    return dp(0, 0)
```

## 五、重叠子问题定性判断

有的读者也许会问，你怎么知道这个问题是个动态规划问题呢，你怎么知道它就存在「重叠⼦问题」呢，这似乎不容易看出来呀？

解答这个问题，最直观的应该是随便假设⼀个输⼊，然后画递归树，肯定是可以发现相同节点的。这属于定量分析。

其实不⽤这么⿇烦，下⾯我来教你定性分析，⼀眼就能看出「重叠⼦问题」性质。

先拿最简单的斐波那契数列举例，我们抽象出递归算法的框架：

```python
def fib(n): 
    fib(n - 1) #1 
    fib(n - 2) #2
```

看着这个框架，请问原问题 f(n) 如何触达⼦问题 f(n - 2) ？有两种路径，⼀ 是 f(n) -> #1 -> #1, ⼆是 f(n) -> #2。前者经过两次递归，后者进过⼀次递归⽽已。**两条不同的计算路径都到达了同⼀个问题，这就是「重叠⼦问题」。**

&#x20;⽽且可以肯定的是，**只要你发现⼀条重复路径，这样的重复路径⼀定存在千 万条，意味着巨量⼦问题重叠**。

同理，对于本问题，我们依然先抽象出算法框架：

```python
def dp(i, j): 
    dp(i, j + 2) #1 
    dp(i + 1, j) #2 
    dp(i + 1, j + 1) #3
```

提出类似的问题，请问如何从原问题 dp(i, j) 触达⼦问题 dp(i + 2, j + 2) ？⾄ 少有两种路径，⼀是 dp(i, j) -> #3 -> #3，⼆是 dp(i, j) -> #1 -> #2 -> #2。因此，本问题⼀定存在重叠⼦问题，⼀定需要动态规划的优化技巧来处理。

## 六、最后总结

通过本⽂，你深⼊理解了正则表达式的两种常⽤通配符的算法实现。其实点 号「.」的实现及其简单，关键是星号「\*」的实现需要⽤到动态规划技巧， 稍微复杂些，但是也架不住我们对问题的层层拆解，逐个击破。另外，你掌握了⼀种快速分析「重叠⼦问题」性质的技巧，可以快速判断⼀个问题是否 可以使⽤动态规划套路解决。

回顾整个解题过程，你应该能够体会到算法设计的流程：从简单的类似问题 ⼊⼿，给基本的框架逐渐组装新的逻辑，最终成为⼀个⽐较复杂、精巧的算 法。所以说，读者不必畏惧⼀些⽐较复杂的算法问题，多思考多类⽐，再⾼ ⼤上的算法在你眼⾥也不过⼀个脆⽪。

### c++代码

```cpp
typedef map<pair<int,int>,bool> pmap;
class Solution {
    pmap dict;// 备忘录 
    /*也可以使用vector<vector<int> > dict(ns,vector<int>(np,-1));
    0  表示 false
    1  表示 true
    -1 表示没有遍历到*/
public:
    bool isMatch(string s, string p) {
        return dp(s,p,0,0);
    }

    bool dp(string s, string p,int i, int j){
        int ns = s.length();
        int np = p.length();
        bool ans;
        pmap::iterator it = std::find_if(
            dict.begin(),dict.end(),[=](const pair<pair<int,int>,bool> && a){
                pair<int,int> location = a.first;
                return location.first == i && location.second == j;
            });
        if(it != dict.end()) return it->second; //在备忘录里，直接返回

        if(j==np) return i ==ns;
        bool first = i<ns && (p[j]==s[i] || p[j]=='.'); //匹配当前
        if(j<=np - 2 && p[j+1]=='*'){  //发现'*'
            ans = dp(s,p,i,j+2) //'*'匹配0个
            || (first&&dp(s,p,i+1,j));    //'*'匹配一个    
        }else{
            ans = first && dp(s,p,i+1,j+1); //没有'*'，都往后挪一个
        }
        dict.insert(make_pair(make_pair(i,j),ans));
        return ans;
    }
};
```
