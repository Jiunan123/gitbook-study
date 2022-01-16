# rebase 



## Rebase 后怎么回退到rebase 之前

```bash
# 可以查看全部的操作记录
git reflog
#回退到rebase之前的版本
git reset --hard xxx 
#撤销rebase
git rebase --abort
```





