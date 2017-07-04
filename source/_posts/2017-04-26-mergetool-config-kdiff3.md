---
title: 'mergetool config:kdiff3'
date: 2017-04-26 09:58:20
categories:
- Git
---

1. 查看git config的配置，是否已经配置了merge工具
	```
    git config --global -l
    ```
2. 如果没有配置merge tool
	```
    git config --global --add merge.tool kdiff3
    git config --global --add mergetool.kdiff3.path "D:/Program Files/KDiff3/kdiff3.exe"
    ```
3. 如果已经配置需要需要修改安装位置
	```
    git config --global merge.tool=kdiff3
    git config --global mergetool.kdiff3.path="D:/path/.."
    ```

