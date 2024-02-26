# Trie树（字典树，单词查找树）

![](<../.gitbook/assets/image (59).png>)

## 例题引入

[【洛谷P2580】于是他错误的点名开始了](https://www.luogu.com.cn/problem/P2580)

题目大意：给出n个单词，有m个询问，每次给出一个单词，如果这个单词出现过且是第一次出现，输出“OK”，如果这个单词没有出现过，输出“WRONG”，如果这个单词出现过但不是第一次出现，输出“REPEAT”，其中n≤10000，m≤100000，每个单词长度l≤50（洛谷 P2580）

对于这道题，暴力是很好做的，每次询问都枚举一下n个单词，再判断一下是否符合题意

但是这样做的时间复杂度是 $$O（nml）$$ ，很明显会超时

这个时候我们就要用到Trie树了，Trie树每次插入和查询的复杂度都为 $$O（l）$$ ，总复杂度为 $$O（（n+m）*l）$$&#x20;

## 介绍

**Trie树**是一种**树形结构**，是一种**哈希树**的变种。典型应用是用于**统计**，**排序**和**保存大量的字符串**（但不仅限于字符串），所以经常被**搜索引擎**系统用于文本词频统计。它的优点是：利用字符串的公共前缀来**减少查询时间**，最大限度地**减少无谓的字符串比较**，查询效率比哈希树高。

## 基本思想

那么首先，Trie树长什么样子呢？

![](<../.gitbook/assets/image (63).png>)

上图就是由单词at，bee，ben，bt，q组成的Trie树

很容易可以看出，每个字母的父亲节点就是它的前一个字母

Trie树的三个性质：

> 1. **根节点不包含字符，除根节点外每一个节点都只包含一个字符**
> 2. **从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串**
> 3. **每个节点的所有子节点包含的字符都不相同**

这样看来，对于一个长为l的单词，无论是插入还是查询都是O（l）的时间复杂度

我习惯于用结构体来存储Trie树：

```cpp
struct Trie
{
	int son[26];   //son[i]记录的当前节点的子节点 
	int num;       //num是当前这个单词在查询中出现的次数(题目要求) 
}a[1000005];       //其实也不用开到这么大,但我一般为了保险都喜欢开大一点 

```

那么，接下来就是介绍如何**插入**和**查询**了

### **插入**

插入操作就是将单词的每个字母都逐一插入Trie树，插入前看这个字母对应的节点是否存在，若不存在就新建一个节点，否则就共享那一个节点，还是以下图为例：

![](<../.gitbook/assets/image (37).png>)

假如说我们要在原Trie树中新插入一个单词and，那我们的操作为：

> 1. 插入第一个字母a，发现根节点存在子节点a，则共享节点a
> 2. 插入第二个字母n，发现节点a不存在子节点n，则新建子节点n
> 3. 插入第三个字母d，发现节点n不存在子节点d，则新建子节点d

代码如下：

```cpp
char x[15];        //x是当前的单词 
int t=0;           //t是节点的编号 
void build_trie()
{
	int i,l,p=0;   //p是当前字母的编号 
	l=strlen(x);
	for(i=0;i<l;++i)
	{
		if(a[p].son[x[i]-'a']==0)      //如果这个子节点不存在 
		  a[p].son[x[i]-'a']=++t;      //新建一个子节点 
		p=a[p].son[x[i]-'a'];          //插入下一个字母 
	}
}
```

### **查询**

查询操作和插入操作其实差不多，就是在Trie树中找这个单词的每个字母，若找到了就继续找下去，若没有找到就可以直接退出了，因为若没找到就说明没有这个单词，还还还是以下图为例：

![](<../.gitbook/assets/image (19).png>)



假如说我们要在原Trie树上查询单词and是否存在，那我们的操作为：

> 1. 查询第一个字母a，发现根节点存在子节点a，则继续查询n
> 2. 查询第二个字母n，发现节点a不存在子节点n，则直接退出并返回0

```cpp
char x[15];        //x是当前的单词 
int get_answer()
{
	int i,l,p=0;   //p是当前字母的编号 
	l=strlen(x);
	for(i=0;i<l;++i)
	{
		if(a[p].son[x[i]-'a']==0)      //如果这个子节点不存在 
		  return 0;                    //直接退出并返回0 
		p=a[p].son[x[i]-'a'];          //查询下一个字母 
	}
	a[p].num++;                        //这个单词的查询次数加一(题目要求) 
	return a[p].num;                   //返回它的查询次数 
}
```

## 复杂度分析

Trie树其实是一种用空间换时间的算法，前面也提到过，它占用的空间一般很大，但时间是非常高效的，插入和查询的时间复杂度都是 $$O（l）$$ 的，总体来说还是很优秀的

## 完整代码

下面是例题的完整代码（洛谷 P2580）：

```cpp
#include<cstdio>
#include<cstring>
#include<algorithm> 
using namespace std;
struct Trie
{
	int son[26];
	int num;
}a[1000005];
int t=0;
char x[15];
void build_trie()
{
	int i,l,p=0;
	l=strlen(x);
	for(i=0;i<l;++i)
	{
		if(a[p].son[x[i]-'a']==0)
		  a[p].son[x[i]-'a']=++t;
		p=a[p].son[x[i]-'a'];
	}
}
int get_answer()
{
	int i,l,p=0;
	l=strlen(x);
	for(i=0;i<l;++i)
	{
		if(a[p].son[x[i]-'a']==0)
		  return 0;
		p=a[p].son[x[i]-'a'];
	}
	a[p].num++;
	return a[p].num;
}
int main()
{
	int n,m,i,ans;
	scanf("%d",&n);
	for(i=1;i<=n;++i)
	{
		scanf("%s",x);
		build_trie();
	}
	scanf("%d",&m);
	for(i=1;i<=m;++i)
	{
		scanf("%s",x);
		ans=get_answer();
		if(ans==1)  printf("OK\n");
		if(ans==0)  printf("WRONG\n");
		if(ans>=2)  printf("REPEAT\n");
	}
    return 0;
}
```

{% hint style="info" %}
## 详细例子：[字典树(Trie树)实现与应用](https://www.cnblogs.com/xujian2014/p/5614724.html)
{% endhint %}
