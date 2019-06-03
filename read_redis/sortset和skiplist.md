https://blog.csdn.net/w372426096/article/details/77800642

首先介绍跳表:
[跳表论文](http://tuvalu.santafe.edu/~aaronc/courses/5454/readings/Pugh_SkipLists_ProbabilisticAlternativeToBalancedTrees.pdf)
跳表的查询复杂度`O(logN)`
```
Search(list , key){

   // 循环不变式   key >= x[i]
   x := list->head
   for i = list->level  downto 1
      while key < x->key
          x := x->next[i]
   
}
```
