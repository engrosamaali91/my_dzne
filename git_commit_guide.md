This is applicable when you mistakenly git add . everything, this in returns not only add the modified file but also the binary files that are not necessary.

```shell
git status

git log --oneline 

git commit -m "binary files removed"

git log --oneline 

git branch 

git diff 7dac 9cbe

git rm --cached FastSurferCNN/config/__pycache__ checkpoints FastSurferCNN/data_loader/__pycache__
git rm --cached -r FastSurferCNN/config/__pycache__ checkpoints FastSurferCNN/data_loader/__pycache__

git status 

git commit -m "more files removed"

```
