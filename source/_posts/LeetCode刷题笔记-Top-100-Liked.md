---
title: LeetCode刷题笔记--Top 100 Liked
date: 2022-03-02 21:36:46
tags: index_img: /img/LeetCode_Sharing.png
banner_img: /img/h4ear4i3g4q7r04utgpm.png
tags: [C++,牛课网]
categories: 实习
---

### 392. Is Subsequence

#### Description

> A **subsequence** of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., `"ace"` is a subsequence of `"abcde"` while `"aec"` is not).

Example 1:

```
Input: s = "abc", t = "ahbgdc"
Output: true
```

#### Idea

>   Use two different index as two pointers, first one represents **s** pointer (like s[index]), second one represents **t** pointer (like t[index]).
>
>   Then we want to analyse if **s** is the subsequence of **t** by check their content one by one: For instance, s = "abc", t = "ahbgdc", s[0] is "a" equals to t[0], so we increase the index and check the next letter s[1] and t[1]; now t[1] doesn't equals to s[1], so increase t's index and check if next letter in **t** equals to **s[1]**.

#### Code

```c++
class Solution {
public:
    bool isSubsequence(string s, string t) {
      	// initialize two pointers
        int s_index = 0;
        int t_index = 0;
        // Check the letter one by one
        while ( t_index < t.size() ) {
            if ( s[s_index] == t[t_index]) {
                s_index++;
                t_index++;
            } else {
                t_index++;
            }
        }
        if ( s_index == s.size()) {
            return true;
        } else {
            return false;
        }
    }
};
```

