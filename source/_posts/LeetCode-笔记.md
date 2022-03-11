---
title: LeetCode 笔记
date: 2022-03-08 20:12:17
tags: 
  - C++
  - 算法
  - 笔记
categories: 算法
index_img: https://gitee.com/Fantastic-Feng/picgo/raw/master/202203111618917.jpg
math: true
---

## 1. 两数之和

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案
 ![image-20220302182050779](https://gitee.com/Fantastic-Feng/picgo/raw/master/202203021820930.png)

链接：https://leetcode-cn.com/problems/two-sum

### 解法：

常规思路是使用两个for循环，遍历数组中的组合方式，返回满足结果的答案；

时间复杂度是$O(n^2)$;

核心代码

```c++
for (int i = 0; i < numslength; i++)
{
    for (int j = i; j < numslength; j++)
    {
        if(nums[i]+nums[j]==target && i!=j)
        {
            res.emplace_back(i);
            res.emplace_back(j);
        }
    }
}
```

第二种解法，使用map数据结构，<key,value>=<nums[i],index>;遍历数组，使用count();计算target-nums[i]的个数，使用map在$O(1)$的时间复杂度内找到taget-nums[i]的index;

因为count()函数时间复杂度为$O(\log ^{n})$,总的时间复杂度为$O(n\log{n})$;

```c++
 vector<int> twoSum(vector<int>& nums, int target){
    vector<int> res;
    map<int, int> M;
    int numslength = nums.size();
    for (int i = 0; i < numslength;++i){
        M.insert({nums[i], i});
    }
    for(int i = 0; i < numslength;++i)
    {
        int numsOfOther = M.count(target - nums[i]);
        if(numsOfOther>0&&(M[target - nums[i]]!=i)){
            res.emplace_back(i);
            res.emplace_back(M[target - nums[i]]);
            break;
        }
    }
}
```



## 2.两数相加

给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

 ![image-20220302193756590](https://gitee.com/Fantastic-Feng/picgo/raw/master/202203021937657.png)
链接：https://leetcode-cn.com/problems/add-two-numbers

### 解题

难点在于返回的是一个指针

建立一个头节点，使用while循环，遍历两个数组，对应数字相加，mod 10放入新链表，、10与下一组累加；同时在循环外判断最后一个数在/10是否大于0，如果大于0，新加一个节点；

```c++
struct ListNode
{
    int val;
    ListNode *next;
    ListNode() : val(0), next(nullptr) {}
    ListNode(int x) : val(x), next(nullptr) {}
    ListNode(int x, ListNode *next) : val(x), next(next) {}
};

ListNode *addTwoNumbers(ListNode *l1, ListNode *l2)
{

    ListNode *prehead = new ListNode(0);
    ListNode *cur = prehead;

    int t = 0;
    while (l1 || l2)
    {
        if (l1)
            t += l1->val, l1 = l1->next;
        if (l2)
            t += l2->val, l2 = l2->next;  // 一般情况下 t += (A[i] + B[i])后是一个0-19大的数字，个位push到当前位，而十位只有0和1作为进位继续后面的加法
        cur->next = new ListNode(t % 10); // t % 10 是 t的个位
        cur = cur->next;
        t /= 10; // t/=10，计算是否有进位，并更新t, 在下一轮继续 t += (A[i] + B[i])
    }
    if (t)
    {
        cur->next = new ListNode(t);
    } // 别忘了如果最后一位加法完成后，还得考虑进位
    return prehead->next;
}
```



## 3. 无重复字符的最长字串

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。

 <img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203021951814.png" alt="image-20220302195158780" style="zoom:100%;" />



 <img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203021952737.png" alt="image-20220302195217708" style="zoom: 80%;" />

### 解题：

使用滑动数组解决

定义一个数组numOfCha用来记录每种字母与空格的数量，定义leftIndex标记子串的左界

使用for循环遍历字符串：如果遍历到的字符已出现次数为0,子串向右加一，numOfCha更新，计算长度(i-letIndex+1)，更新最大长度；否则不断弹出最左侧的字符，leftIndex++,相当于向右滑动；

```c++
int lengthOfLongestSubstring(string s)
{
    int leftIndex=0, maxLength=0;
    int len = s.size();
    int numOfCha[100];
    memset(numOfCha, 0, sizeof(numOfCha));
    set<char> m;
    for (int i = 0; i < len;i++)
    {
      while (numOfCha[s[i]-' ']!=0)
      {
          numOfCha[s[leftIndex] - ' ']--;
          leftIndex++;
      }
      maxLength = max(maxLength, i - leftIndex + 1);
      numOfCha[s[i] - ' ']++;
    }
    return maxLength;
}
```



## 5. 最长回文子串

给你一个字符串 `s`，找到 `s` 中最长的回文子串

 ![image-20220302214119440](https://gitee.com/Fantastic-Feng/picgo/raw/master/202203022141480.png)

### 解法：

- 扩展中心

  我们知道字符串一定是对称的，所以我们可以每次循环的时候选择一个中心，进行左右扩展，判断新扩展的字符是否相等。

   ![image-20220304185620795](https://gitee.com/Fantastic-Feng/picgo/raw/master/202203041856092.png)
  
  因为存在奇数的字符串或者偶数的字符串，所以我们需要从一个字符开始扩展，或者两个连续的字符开始扩展，所以共有n + (n-1)个中心。
  
  时间复杂度：O($n^2$）O($n^2$）。
  
  空间复杂度：O(1）O(1）。
  
  ```c++
  int expendAroundCenter(string s, int lef, int rig)
  {
      int subLef = lef;
      int subRig = rig;
      // 左界应该在大于等于0，右界应当小于字符串的长度，同时新扩展的两个字符应当相等
      while ((subLef >= 0) && (subRig < s.length()) && (s[subLef] == s[subRig]))
      {
          subLef--;
          subRig++;
      }
      //返回的是以s[lef,rig]为中心的回文子串长度
      return subRig - subLef + 1;
  }
  
  //主函数
  string longestPalindrome(string s)
  {	
      //如果S是空，则返回空
      if (s == "" || s.length() < 1)
          return "";
      //用来标记最长回文子串的左右界
      int strStart = 0, strEnd = 0;
      
      int strLen = s.length();
  	//遍历每一个字符
      for (int i = 0; i < strLen; i++)
      {	
          //以该字符为中心，或者连续两个字符为中心
          int len1 = expendAroundCenter(s, i, i);
          int len2 = expendAroundCenter(s, i, i + 1);
          int len = max(len1, len2);
          //如果长度大于我们标记的，更新左右界
          if (len > strEnd - strStart + 1)
          {
              strStart = i - (len - 1) / 2;
              strEnd = i + len / 2;
          }
      }
      return s.substr(strStart, strEnd - strStart + 1);
  }
  ```
  
  

## 8. 字符串转整数

请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。

函数 myAtoi(string s) 的算法如下：

1. 读入字符串并丢弃无用的前导空格
2. 读入下一个字符，确定正负号，若不存在假定为正
3. 读入数字转化为整数，无数字则为0
4. 若溢出int,则截断整数

---

输入: s=“   -42  akkk”

输出: -42



输入: s="-2147483648"

输出: -2147483648



输入: s=“ 2147483648”

输出: 2147483648



---



链接：https://leetcode-cn.com/problems/string-to-integer-atoi

### 解法：

思路较为清晰和明确，整体是遍历一遍字符串

首先去除开头的空格，index++;

如果index==len,全是空格，则返回0；

之后判断正负，使用sign标定

转化数字时因为只能使用32位，所以判断时应该与INT_MAX/10比较，因为INT数字范围是$\left[ -2^{32},2^{32}-1 \right]$，所以在判断个位数时应该使用大于

```c++
int myAtoi(string s)
{
    int len = s.length();
    int index = 0, sign = 1, res = 0;
    //如果是空格，索引向后
    while (s[index] == ' ' && index < len)
    {
        index++;
    }
    if (index == len)
    { //整个字符串都是空格
        return 0;
    }
    else
    {
        if (s[index] == '-')
        { //是否为负数
            sign = -1;
            index++;
        }
        else if (s[index] == '+')
        { //是否为正数
            index++;
        }
        while (index < len && s[index] <= '9' && s[index] >= '0')
        {
            int curDigit = s[index] - '0';
            if (res > INT_MAX / 10 || ((res == INT_MAX / 10) && (curDigit > INT_MAX % 10)))
            {
                return sign==1?INT_MAX:INT_MIN;
            }
            else
            {
                res = res * 10 + curDigit;
            }
            index++;
        }
        return sign * res;
    }
}
```
