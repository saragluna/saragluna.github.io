---
layout:     post
title:      "Morris Traversal"
subtitle:   "Binary Tree Inorder Traversal"
date:       "2019-03-28 12:00:00"
author:     "Sarag"
header-img: "img/post-bg-2019.jpg"
tags:
    - Algorigthms
---



> [Explanation from StackOverflow](https://stackoverflow.com/questions/5502916/explain-morris-inorder-tree-traversal-without-using-stacks-or-recursion)

If I am reading the algorithm right, this should be an example of how it works:

```cpp
     X
   /   \
  Y     Z
 / \   / \
A   B C   D
```

First, `X` is the root, so it is initialized as `current`. `X` has a left child, so `X` is made the rightmost right child of `X`'s left subtree -- the immediate predecessor to `X` in an inorder traversal. So `X` is made the right child of `B`, then `current` is set to `Y`. The tree now looks like this:

```cpp
    Y
   / \
  A   B
       \
        X
       / \
     (Y)  Z
         / \
        C   D
```

`(Y)` above refers to `Y` and all of its children, which are omitted for recursion issues. The important part is listed anyway. Now that the tree has a link back to X, the traversal continues...

```cpp
 A
  \
   Y
  / \
(A)  B
      \
       X
      / \
    (Y)  Z
        / \
       C   D
```

Then `A` is outputted, because it has no left child, and `current` is returned to `Y`, which was made `A`'s right child in the previous iteration. On the next iteration, Y has both children. However, the dual-condition of the loop makes it stop when it reaches itself, which is an indication that it's left subtree has already been traversed. So, it prints itself, and continues with its right subtree, which is `B`.

`B` prints itself, and then `current` becomes `X`, which goes through the same checking process as `Y` did, also realizing that its left subtree has been traversed, continuing with the `Z`. The rest of the tree follows the same pattern.

No recursion is necessary, because instead of relying on backtracking through a stack, a link back to the root of the (sub)tree is moved to the point at which it would be accessed in a recursive inorder tree traversal algorithm anyway -- after its left subtree has finished.