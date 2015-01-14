title: 使用chmod如何给文件夹递归授予权限
date: 2015-01-14 12:52:37
tags: [linux,chmod,递归,权限]
---
```bash
find /path -type f -exec chmod 644 {} \; 修改文件权限
find /path -type d -exec chmod 755 {} \;修改文件夹权限
```

- `/path` , -type 根据类型和路径遍历文件以及子文件夹
- `-exec` 每遍历一步就执行一次此参数后的命令
- `{}` 代表前面所遍历到的文件或文件夹
- `\;` 代表-exec命令的结束符号，-exec命令的固定格式