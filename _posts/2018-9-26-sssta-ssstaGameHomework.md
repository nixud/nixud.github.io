---
layout: post
title:  "SSSTA-Game组作业及答案"
date:   2018-09-26
excerpt: "SSSTAGame组18级各次活动的作业以及答案（长期更新）"
tag:
- sssta
comments: true
---

### 第一次作业

##### 入门

输入三个整数x,y,z，把这三个数从小到大输出。

要求使用C#。

##### 稍微难一点的入门

给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。

你可以假设每个输入只对应一种答案，且同样的   元素不能被重复利用。

示例:

给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9

所以返回 [0, 1]

要求使用C#。

#### 答案

1.
```
public int[] TwoSum(int[] a)
        {
            Array.Sort(a);
            return a;
        }
```

2.
```
//以下为暴力法
public int[] TwoSum(int[] nums, int target)
{
    for (int i = 0; i < nums.Count(); i++)
    {
        for (int u = i + 1; u < nums.Count(); u++)
        {
            if (nums[i] + nums[u] == target)
            {
                return new int[] { i, u };
            }
        }
    }

    throw new Exception("No two sum solution");
}
```