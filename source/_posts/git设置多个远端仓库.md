---
title: git设置多个远端仓库
date: 2026-01-27 10:08:37
tags:
---
### 修改 origin（最推荐，无感操作）

这种方案是给默认的 origin 添加额外的推送地址。当你执行 git push 时，Git 会遍历所有配置的推送地址。

**注意：** 一旦你手动设置了 pushurl，Git 就会忽略默认的 url（即你原本 clone 下来的那个地址）作为推送目标。所以，你需要把**原本的地址**和**新的地址**都加进去。

假设你现在的 origin 指向 GitHub，你想增加 Gitee。

**操作步骤：**

1. **查看当前远程地址：**
    
    ```bash
    git remote -v
    # 输出示例:
    # origin  git@github.com:user/repo.git (fetch)
    # origin  git@github.com:user/repo.git (push)
    ```
    
2. **设置第一个推送地址（原本的 GitHub）：**
    
    ```bash
    git remote set-url --add --push origin git@github.com:user/repo.git
    ```
    
3. **设置第二个推送地址（新的 Gitee）：**
    
    
    ```bash
    git remote set-url --add --push origin git@gitee.com:user/repo.git
    ```
    
4. **再次验证：**
    
    ```bash
    git remote -v
    ```
    
    此时你应该会看到 origin 有**一个** (fetch) 地址，但有**两个** (push) 地址：
    
    
    ```bash
    origin  git@github.com:user/repo.git (fetch)
    origin  git@github.com:user/repo.git (push)
    origin  git@gitee.com:user/repo.git (push)
    ```
    
5. **使用：**  
    以后只需执行：
    
    ```bash
    git push
    ```
    
    Git 就会依次把代码推送到这两个仓库。