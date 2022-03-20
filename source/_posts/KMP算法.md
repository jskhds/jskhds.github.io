---
title: KMP算法
date: 2021-10-18 14:51:13
tags: KMP算法

---

#### 1.概述：

KMP算法主要用在字符串匹配上，需要一个前缀表和一个next数组

<!-- more -->

#### 2.过程：

以模板字符串 aabaaf 和目标字符串 aabaabaafa 为例：

1）如果暴力匹配，那么在 f 匹配不上时，就需要从头开始匹配，效率很低

2）如果是KMP算法，模板字符串匹配到 f 时， 不从头开始，而是从 b 开始匹配。思想非常简单，我们可以发现，对于 aabaabaafa 在第二个 b 我们匹配不上了，但是这个b前面的aa和模板字符串的开头aa是一样的，所以从aa后面一个值开始匹配是效率最高的。

动图如下：

 ![](D:\program\blog\source\images\KMP精讲1.gif)

#### 3.前缀表

现在有一个问题是，在字符不匹配时，我们怎么知道下一步该从哪里开始呢？这里就需要用到前缀表了。

###### 1）先说一下什么是前缀和后缀：

前缀：不包括最后一个字符，包括第一个字符的所有连续子字符串。

eg：aabaaf的所有前缀：a, aa, aab, aaba, aabaa 不包括aabaaf

后缀：与前缀刚好相反 

eg：aabaaf的所有后缀：f, af, aaf, baaf, abaaf  不包括aabaaf

###### 2）前缀表：记录当前索引 i 之前（包括 i ）的字符串中，有多大长度的相同前后缀。

aabaaf的前缀表 

| 0    | 1    | 2    | 3    | 4     |
| ---- | ---- | ---- | ---- | ----- |
| a    | aa   | aab  | aaba | aabaa |
| 0    | 1    | 0    | 1    | 2     |

单纯得到这个表还是很简单的，要么把字符串的前后缀都写出来，看交集最长为多少，要么就直接看字符串前后重合的字符有多少个。关键是要理解这个表的意思。

######  3)前缀表的含义

前面已经说过，碰到不匹配的字符串时，我们不能浪费掉之前已经匹配过的字符串，我们跳过索引 i 之前已经匹配好了的字符串，从这个字符串的后面一个开始匹配。

目标字符串从索引5的 b 开始不匹配，也就是说b以前的都是匹配的，而我们又知道在模板字符串里，同样索引位置的前两个字符在开头有重复，所以，我们就可以跳到第三个（索引2）开始下一次匹配。这也是KMP算法对暴力算法的优化所在于，它不再从头开始。

#### 4.得到next数组

```js
 const getNext = (s)=>{
        let next = [];
        let j = 0;
        next.push(j);
        for(let i = 1;i<s.length;++i){
        //注意这里是while 不是 if 因为j是连续回跳，直到到了索引0或者跳到了指定位置
            while(j>0&&s[i]!==s[j]){
                j = next[j-1]
            }
            if(s[i]===s[j])
                j++;
            next.push(j);
        }
        return next;
    }
```

#### 5.例题

###### leetcode459 重复的字符子串 https://leetcode-cn.com/problems/repeated-substring-pattern/

给定一个非空的字符串，判断它是否可以由它的一个子串重复多次构成。给定的字符串只含有小写英文字母，并且长度不超过10000。 

```js
示例：
输入: "abab"
输出: True
解释: 可由子字符串 "ab" 重复两次构成。

输入: "aba"
输出: False
```

```js

// KMP
var repeatedSubstringPattern = function(s) {
    if(s.length === 0) return false;
    const getNext = (s)=>{
        let next = [];
        let j = 0;
        next.push(j);
        for(let i = 1;i<s.length;++i){
            while(j>0&&s[i]!==s[j]){
                j = next[j-1]
            }
            if(s[i]===s[j])
                j++;
            next.push(j);
        }
        return next;
    }
    let next = getNext(s);
    if(next[next.length-1]!==0 && s.length%(s.length - next[next.length - 1])===0)
        return true;
    return false;
};
```

###### leetcode28 实现 strStr() 函数 https://leetcode-cn.com/problems/implement-strstr

给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串出现的第一个位置（下标从 0 开始）。如果不存在，则返回  -1 。

 

```js
var strStr = function(haystack, needle) {
    if(needle.length === 0) return 0;
    const getNext = (needle)=>{
        let next = [];
        let j = 0;
        next.push(j);
        for(let i = 1;i<needle.length;++i){
            while(j>0&&needle[i]!==needle[j]){
                j = next[j-1]
            }
            if(needle[i]===needle[j])
                j++;
            next.push(j);
        }
        return next;
    }
    let next = getNext(needle);
    let j = 0;
    for(let i = 0;i<haystack.length;++i){
        while(j>0&&haystack[i]!==needle[j])
            j = next[j-1];
        if(haystack[i]===needle[j])
            j++;
        if(j === needle.length)
            return (i-needle.length+1)
    }
    return -1;
};
```

