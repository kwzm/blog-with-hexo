---
title: git revert后导致合并代码丢失
date: 2019-01-18 17:07:04
categories: [技术, 前端]
tags: [git]
---

## 起因

我有一个开发分支 antd3.x 和一个主分支 develop，我在合并 antd3.x 到 develop 的时候发现有些修改没有合并进来。

<!-- more -->

## 查找问题

然后就去网上查，发现这篇文章[git 合并丢失代码问题分析与解决](https://www.jianshu.com/p/603186352605)给我了一些启发。
　　其中说到 git merge 的原理是三方合并，简单来说就是假设我有 a 和 b 两个分支，我要合并 b 到 a，这个时候 git 其实还会去找到 a 和 b 的最近的父节点 c，将 c 作为基础的分支，然后对 abc 进行比较，如果有一个文件 xxx.js，xxx.js 的内容 abc 三个分支上同一行都不一样那么就会报 conflict，因为 git 也不知道该保留谁的代码，就会让你自行决断；如果只有一个分支比如 b 上面的 xxx.js 的同一行代码和 c 分支上的不一样那么 git 就会自作主张的认为应该保留 b 上面的修改，看到这我就有了方向，我应该去找我此次合并的两个分支的那个最近的父节点，问题应该就出在他上面。
　　通过执行 git merge-base antd3.x develop 找到了他俩 base 节点 2344a88，然后查看该节点的代码发现那些没有合并进来的修改已经存在于这个 base 节点上了，怪不得进行三方合并的时候没有合并进来，因为 git 发现 develop 分支上对比 base 分支没有这些修改，于是这些修改就被删除掉了。但是到这一步我又有疑问了，我没有手动的删除过 develop 上的这些修改，为啥这些修改会没有了呢，于是我查看 log 发现我曾经通过 git revert 撤销过一个合并，而这个合并恰巧就是那个修改的内容，到这块问题基本就清楚了，下面我总结一下。

## 总结

让我们从头捋一下这个问题的前因后果：

- 我有三个分支 develop、antd3.x 和 std-08，其中 antd3.x 是根据 develop 拉取的，std-08 是根据 antd3.x 拉取的
- 此时我想要合并 antd3.x 到 develop 分支上，执行代码 git merge --no-ff antd3.x
- 发现 antd3.x 上面的有些修改（x）没有合并到 develop 上
- 通过 git merge-base antd3.x develop 找到了他俩 base 节点 2344a88
- 发现 base 节点上面存在修改 x
- 通过 git log 发现 develop 分支上曾经执行过 git revert
- 本来当时是想把 std-08 合并到 antd3.x 上结果合并到了 develop 上，也就是此时修改 x 合并到了 develop 分支上
- 然后执行了 git revert 撤销了此次合并，并正确合并 std-08 到 antd3.x 上，这就导致了 antd3.x 和 develop 的共同父节点变成了 std-08 的最后一次提交 2344a88
- 所以后续再想合并 antd3.x 到 develop 上时，进行三方合并，发现 base 节点上的修改在 develop 上面被删除了，所以合并的结果就是删除这些修改，但是实际上这些修改我们是想保留的
  解决方案

## 解决方案有两个：

1. 在 develop 上执行 git reset commitId 到合并 std-08 之前的那次提交然后 git push -f origin ，这样 develop 的 commit 历史就不包含那次合并了
   　　 2. 在 develop 上执行 git revert HEAD~ 把之前 revert 的再 revert 掉，这个我没有试过

## 思考

其实这个教训充分暴露了我对 git revert 和 git reset 的区别不甚了解，可以看下这篇文章《git revert 用法》，对它俩的区别解释的很清楚，其实就是 git revert 的撤销原理就是删除掉之前的提交然后执行 commit，这样所有的 commits 都会保留下来，也就埋下了隐患，在我的场景里就是在 git 合并的时候寻找的 base 节点就是 develop revert 之前的那个 commit，如果是 git reset 就不会保留这个 commit，也就不会把它作为 base 节点，合并的时候就不会有问题。我看了下网上还有另一种解释就是执行完 git revert 后，git 就认为你不在需要这些修改，以后再合并的时候如果有这样的修改要合并，git 就会忽略，我没有验证过这种说法的真实性，如果有人清楚的话，还请不吝赐教。
