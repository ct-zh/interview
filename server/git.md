1. git文件版本，使用顺序，merge跟rebase


2. git回滚
    1. `git reset`: 直接回到指定commit;
    2. `git revert`: 生成一次新的commit;

3. git和svn区别，模型
    git是分布式的,svn不是

4. git pull 做了哪些工作？
    git pull背后是git fetch+git merge：git fetch是将远程仓库新增加的内容拉取到本地仓库，git merge是将此内容与本地仓库相同分支的内容进行合并

