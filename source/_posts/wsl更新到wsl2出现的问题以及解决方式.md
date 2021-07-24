---
title: wsl更新到wsl2出现的问题以及解决方式
date: 2020-06-15 10:10:43
tags: [wsl, debug]
category: 计算机技术
---

# wsl更新到wsl2出现的问题以及解决方式

花了很多时间折腾wsl,从用上wsl那一刻起我就以为我用的是wsl2，因为印象中之前确实做版本转换，过程中并没发现任何异样（其实是因为我水平太低了，在我眼前晃悠我也不一定看得到）。直到docker设置里面连接到wsl2那一栏始终是灰色，我才发现自己一直用的是wsl。在切换wsl版本的过程中我遇到两个主要的问题：

## 1.wsl找不到--set-version、wsl --set-default-version

不仅找不到这些，甚至连wsl -l -v都用不了，wsl --help 中不存在这些命令。

### 原因解决方式：

这是因为windows内部版本太低的问题，当时用的版号是1909，shell中用`ver`查看的版本号是18363，而wsl需要19***什么的版本，解决办法是去官网暗网下载Windows易更，把windows更新到最新版本。

## 2.wsl --set-version报错

最开始显示的报错是什么无法修改、创建某个文件，根本找不到解决办法，还好重启之后报错成了：`WslRegisterDistribution failed with error: 0xffffffff`，[github](https://github.com/microsoft/WSL/issues/4364)上给出了解决方案，原来是其它虚拟机占据了进程，我是因为docker在后台运行。

解决办法就是：
```
Get-Process -Id (Get-NetUDPEndpoint -LocalPort 53).OwningProcess
```

```
NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName
 ------    -----      -----     ------      --  -- -----------
     18    26.42      30.45       5.31   16432   0 dockerd
     15    11.11      16.40       0.36    3972   0 svchost
```

然后kill掉上面的进程：

```
Stop-Process -Id 16432 -Force
Stop-Process -Id 3972 -Force
```

现在试试应该就没问题了。