---
title: 'leetcode(140): word break(2)'
date: 2017-07-01 13:48:29
categories:
- leetcode
tags:
- leetcode
- 动态规划
---

# Problem
Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, add spaces in s to construct a sentence where each word is a valid dictionary word. You may assume the dictionary does not contain duplicate words.

Return all such possible sentences.

For example, given
s = `"catsanddog"`,
dict = `["cat", "cats", "and", "sand", "dog"]`.

A solution is `["cats and dog", "cat sand dog"]`.

# Solution
```java
public class Solution {
    public List<String> wordBreak(String s, List<String> wordDict) {
        return dfs(s, wordDict, new HashMap<String, LinkedList<String>>());
    }

    private List<String> dfs(String s, List<String> wordDict, HashMap<String, LinkedList<String>> map){
        if (map.containsKey(s)){
            return map.get(s);
        }

        LinkedList<String> res = new LinkedList<>();
        if (s.length() == 0){
            res.add(""); // 用一个空元素表示可以划分
            return res;
        }
        for (String word: wordDict){
            if (s.startsWith(word)){
                List<String> subList = dfs(s.substring(word.length()), wordDict, map);
                for (String sub : subList){
                    res.add(word + (sub.isEmpty() ? "" :" " + sub));
                }
            }
        }
        map.put(s, res);
        return res;
    }
}
```

```java
import java.util.*;
public class Solution {
    public ArrayList<String> wordBreak(String s, Set<String> dict) {
        ArrayList<String> list = new ArrayList<>();
        Stack<String> stack = new Stack<>();
        dfs(s, dict, s.length(), list, stack);
        return list;
    }
    
    private void dfs(String s, Set<String> dict, int len, ArrayList<String> list, Stack<String> stack){
        if (len <= 0){
            StringBuilder sb = new StringBuilder();
            for (int i = stack.size() - 1; i >= 0; i--){
                sb.append(stack.get(i));
                if (i != 0){
                    sb.append(" ");
                }
            }
            list.add(sb.toString());
        }

        for (int k = len - 1; k >= 0; k--){
            String word = s.substring(k, len);
            if (dict.contains(word)){
                stack.push(word);
                dfs(s, dict, k, list, stack);
                stack.pop();
            }
        }
    }
}
```