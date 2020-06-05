# git基本介绍
* Git 是一个分布式版本控制软件，它的核心本质上是一个键值对数据库。可以向该数据库插入任意类型的内容， 他会返回一个键值，通过该值可以在任意时刻再次检索该内容。
 * git客户端并不只是提取最新版本的文件快照，而是把代码仓库完整的镜像下来
 

##	git的区域
   * 工作区 （就是我们创建git项目的目录）

    * 暂存区  （工作区文件的提交或者回滚都是先通过暂存区，而版本库中除了暂存区之外，文件的提交的存储位置是你当前所在的分支，在创建版本库的时候默认会创建一个主分支（Master），内容存储在.git目录下的index文件夹内）
    
    * 版本库   （最终的代码实现提交到这里 .git目录就是版本库）

## .git（版本库）目录下文件的介绍
   * hooks (钩子函数的一个库 类似于回调函数)
    * info (包含一个全局性的排除文件)
    * objects (目录存储所有数据内容)
    * refs (目录存储指向数据（分支）的提交对象的指针)
    * config (文件包含项目特有的配置选项)
    * description (显示对仓库的描述信息)
    * HEAD (文件目前被检出的分支)
    * logs (日志信息)
    * index (文件保存暂存区的信息，默认没有的)



# git的对象概念
##	Git对象
   **本质上是一个键值对类型的数据，键为值的哈希值，通过以下命令生成git对象**
   * echo "hello" | git hash-object --stdin 
   这句命令返回一个hash值用来标识这句话 但是并没有写到数据库中，内容不一样对应的hash值不一样

  - echo "hello" | git hash-object -w --stdin
    这句命令返回一个hash值用来标识这句话 并写到数据库中  **查看有没有存在 可以通过 find ./ -type f 找对应hash的文件里面的内容是压缩的 通过 git cat-file -p hash注意前面的那两个字母也加上**
   - git cat-file -t hash 查看git对象的类型 blob

  **将新创建的文件添加到git数据库中即生成一个git对象**
  - git hash-object -w ./a.txt
	如果文件更改，git数据库里面不会自动的添加要手动添加过去，添加后， 这时会在添加一个git对象
         
   - git对象不能当作项目的一次快照 只是组成项目的一部分

   ###	存在的问题
   - 记住文件的每一个（版本）对应的hash值并不现实
   - 在git中，文件名并没有被保存，只能通过hash
   - **注意：此时的操作只是针对本地数据库进行操作，不涉及暂存区。**


  ## 树对象
 **本质上是项目的快照一个树对象可以包含多个GIT对象和树对象**，树对象能够解决文件名保存的问题，也允许我们将多个文件组织到一起。

   * 构建树对象，命令如下
     - git update-index --add --cacheinfo 文件模式 SHA-1  test.txt
         - 文件模式为100644 表明这是一个普通文件
         - 文件模式为100755 表明这是一个可执行文件
         - 文件模式为120000 表明这是一个符号连接
         -  --add 因为此前该文件并没有在暂存区中 首次要加add
         -  --cacheinfo 因为要添加的文件在git数据库中,没有位于当前目录下
		- SHA-1 哈希算法,是Git 计算校验和的机制，由 40 个十六进制字符组成的字符串，是基于 Git 中文件的内容或目录结构计算出来的。

    * 暂存区做一个快照生成一个对象放到git数据库中
         - git write-tree
            对象类型是一个树对象，树对象里面的内容是暂存区的快照（项目的快照）
        * 暂存区中文件名字不变 如果改变文件的内容，就会重新生成一个hash

 ### 解决了之前的的问题
   - 不知道hash值对应的是哪一个版本？不知道这个版本的一些基础信息 ？ 
  - 提交树对象完美的解决了上面的问题，**提交树对象-提交树对象的本质就是项目的版本。本质就是给树对象做一层包裹包含项目的基础信息。**
*  commit-tree创建一个提交对象，为此需要指定一个树对象的hash值,以及该提交的父提交对象 
  *  真正代表一个项目的是一个提交对象（数据和基本信息）这是一个链式的！！ 
	 
	
---

#	git基本操作
 ## 初始化git
    * git init （初始化仓库 生成.git文件）
    * git config --global user.name "name"
    * git config --global user.email ***@163.com
    * git config --list
    
## 添加到暂存区
    * git add ./   
    * 首先将工作区（文件）做成git对象放到版本库（.git/objects） 然后再放到暂存区即.git/index文件 但是这里没有生成树对象
    * 相当于 git hash-object -w 文件名（修改了多少个工作目录的文件就要执行多少次）
		git update-index...
	* git ls-files -s  查看暂存区的当前状态

## 添加到版本库
    * git commit -m '提交的信息'，提交并不会清空缓存区。
	* 相当于执行了git write-tree 和 git commit-tree
## 结论
   * 一次完整的项目提交 包括**至少一个提交对象 一个树对象 0或多个git对象**
    * 工作目录中文件只有两种状态 **已跟踪（只要第一次add就跟踪上了） 未跟踪**
    * 已经跟踪的文件还有三种状态 **已提交 已修改 已暂存**
    * 如果一个已经提交的文件再次修改要重新添加到暂存区否则显示已修改状态
    * 如果一个文件暂存完了没有提交前还要在修改 这时会出现一个暂存一个已修改的情况需要重新add

#	分支
简单来说，分支就是指向某个提交对象的指针，你每做一个新的功能时都可以开一个分支，它不影响主线的分支代码。在开始一个git项目时，git都会帮我们默认创建一个分支master。
 ##	分支基本操作
	* git branch 查看分支列表
    * git branch branchName    会在当前的提交对象上创建一个分支
    * git checkout test  将分支转到test上面来
	- NOTE:调用这个功能一定要先调用git status确定没有未提交的暂存或者为暂存的修改目录，不然会污染其他分支.
	因为在切换分支时，工作区目录未被跟踪的文件或者目录也会被带到切换分支的工作区目录中。
    
    * git checkout -b newbranchName 相当于同时执行上面两个方法
	* git checkout commitHash filename 重置暂存区，重置工作目录
	* git checkout filename 重置工作目录
    * git branch -D test  删除分支 不能自己删自己 
	* git branch -d test  合并分支后删除分支 test
	* git branch name commitHash（版本穿梭） 新建一个分支将提交对象指向commitHash。可以利用 git log --online来获取提交对像即版本
    * git log --oneline --decorate --graph --all  查看完整的分支图（没删除前）
    * git config --global alias.lol ‘log --oneline --decorate --graph --all’  将后面一长串的命令设置为lol存储在config文件中，到时候只要调用git lol就可以查看完整的分支图了。
    * git branch -v  查看分支的最后一个提交
  
    * git merge branchName
		快进合并不会产生冲突
		典型合并会产生冲突，典型合并也就是在进行了一次快进合并后之前基于快进合并之前的分支修改的分支和快进合并之后的分支进行合并
		
   ##	结论
   * 分支就是为了保护代码方便更改存在的 假如master里面的提交对象完美了就可以在创建一个分支   添加功能如果可以就可以master合并 不行的话就可以删除这个分支 这样对于master没有影响
    
    * 调用切换分支功能一定要先调用git status确定没有未提交的暂存或者为暂存的修改目录，不然会污染其他分支.

    * 合并分支一定要注意顺序 后面的可能会过期还会存在bug 会产生冲突
       
       

----


# 高级常用命令
	* git config --global alias.代词 "参数"，利用这个高级命令配置可以简化git命令。
	  例如 git config --global alias.lol "log --online --decorate --graph --all",调用 git lol就可以查询提交记录了 
    * git init               初始化
    * git add ./             添加工作区目录文件到暂存区   
    * git commit -m "注释"     提交项目
    * git status              查看文件当前状态
	  - 文件状态分为已跟踪和未跟踪，已跟踪分为已提交和已暂存和已修改。git add命令执行后文件就从未跟踪变为已跟踪状态，然后再根据文件是否修改提交等自动划为对应状态。
	  - 当调用过git add的文件或者文件夹再去修改他的话会在生成一个已修改状态的该文件，如果现在提交只不过提交了之前提交到暂存区的文件，所以在git commit前尽量使用
    * git diff                查看当先做的哪些更新没有暂存
    * git diff --cached|staged        查看哪些已经更新好了准备下次提交
		这两个命令比git status更加详细，查看文件内容
    * git commit -a -m        git自动将已经跟踪过的文件暂存起来一并提交
    * rm yd.txt               删除文件 暂存区里没有文件 版本库多了一个提交对象不过没有这个文件的git对象
    * 删除文件属于修改操作 跟上面的提交步骤一样
	* git rm 相当于先做了rm然后在进行了git add，把修改添加到暂存区，使用了git rm然后git commit就行了
	* git mv同理
    * mv z.txt zx.txt         重新起名字跟已修改一样的操作
    * git log --oneline      查看提交历史记录但是只能看到当前分支的所有提交
	* git reflog 	查看整个项目的HEAD变化过程
    * git checkout --track 别名/分支名  自动创建本地分支，分知名为远程分支名并且与远程跟踪分支绑定
    * git checkout -b 分支名 别名/分支名 效果与上面一样     
	* git branch -vv 查看分支是否有远程跟踪分支绑定
	* git push use(别名) master（分支）将你在当前分支所提交的改变推到别名为use的远程连接远程仓库的master分支上
	* git pull  use(别名) master（分支） 将别名为use的远程仓库的master分支的代码拉到当前所在分支并且合并它们。


# 存储
    * 解决的问题 不想过多的创建提交
	* git stash 存储
    * git stash list 查看存储
    * git stash apply  拿出栈顶的元素 但是不会消除栈顶元素
    * git stash drop 名字 取出栈顶元素
	* git stash pop 相当于apply加上drop。更加常用！！
  
----
# 后悔药
    * 工作区撤回在工作目录中的修改
      * git checkout -- filename  本质是相当于重置
    * 暂存区撤回自己的暂存
      * git reset HEAD filename
    * 提交区注释写错了修改注释
	  * 先操作工作目录和暂存区，然后git add后提交。最后
       git commit --amend，修改或者不修改提交注释即可完成提交区后悔操作。
	*  原理-底层命令实现
		git reset --soft HEAD(其他树对象的HASH对象) 作用类似于上面的git commit --amend，原理就是HEAD带着分支一起移动到前一个树对象。
		git reset --mixed HEAD(其他树对象的HASH对象/文件路径) 会重置暂存区的内容和head带着分支
		git reset --hard HEAD(其他树对象的HASH对象)	作用类似与checkout 会充值暂存区重置工作目录
			他与checkout的共同点：都需要重置暂存区 head 和工作目录
			他与checkout的区别 1.checkout只动HEAD --hard动HEAD并且带者分支一起动2.checkout对工作目录是安全的 --hard是强制覆盖工作目录
	*  恢复到以前的分支
		git branch new-branch-name commitHash
#	打tag
	*	tag就是不会动的分支
	*	创建tag，	git tag name commitHash
	*	git show tagname查看tag之乡的提交对象
	*	git tag -d tagname删除tag
	*	git checkout -b tagname,如果没有-b的话会导致head过来了但是该提交对象没有对应的分支，所以应该在这里直接创建对应的分支。
	*	git push 别名 tag名 传tag到github
# 远程操作github仓库步骤
   1. 先在github上创建一个空的仓库new repository 
    2. 创建本地仓库然后基础设置   git init
    3. 然后给github上面的地址起别名和用户别名
      * git remote add 别名 https://github.com/zhaoyuanmeng/git_to_use.git
	  *	git remote -v 查看当前连接对象别名和对应的url地址
      * git config --list
   4. 注意如果是复制别人的github 要把.git删掉只复制项目部分代码到自己的本地仓库
    5. 注意凭据 本人是可以直接上传的
    6. 检查完毕后推送到远地仓库  git push use(别名) master（分支）
    7. 给其他人开放权限通过github里面的manage access contributor
    8. 获取他热门上传的代码 git fetch use(别名)
    9. 切换成远程跟踪分支 git checkout use/master
    10. 合并远程跟踪分支 git merge use/master 
    11. git pull  获取数据并合并
# 拉取仓库代码
   1. 本地不用创建仓库 直接克隆下来  git clone url
    2. 它自动创建一个别名; 查看别名git remote -v       
    3. 创建新的文件 echo "hello world">test.txt
    4. git add ./   
    5. git commit -m "描述"
    6. git push origin master
    

# 本地分支 远程分支 远程跟踪分支
   * 本地分支是本机电脑的
    * 远程分支是github上对应的分支
    * 远程跟踪分支是本地与远程分支的一个映射
    * 成员克隆远程仓库以后默认本地分支和对应的远程跟踪分支有同步关系
    * 在push的时候会生成对应的远程跟踪分支，当你的本地分支对应的是master时，直接git push可以自动同步到远程分支master，其他的本地分支必须指定别名和远程提交的分支名。
	  git push 别名 分支名。当然也可以绑定远程分支，使用git branch -u 别名/分支名 就可以绑定当前本地分支和对应远程分支，后面就可以直接git push了
    * 在fetch的时候把数据下载到远程跟踪分支里面，或者生成后来生成的远程跟踪分支 git fetch 别名，然后再将远程跟踪分支和本地分支merge一下，就可以将远程仓库的数据合并到本地。
		如果绑定了本地分支和远程跟踪分支，可以直接用git pull，就可以直接将远程数据同步到本地分支。
    * 注意成员开辟新分支提交的时候 经理在fetch的时候要创建对应的分支不用加别名
    * 主分支和远程跟踪分支自动绑定的功能(默认情况下push的时候)
    * 建立同步关系 git branch -u (远程跟踪分支) 注意要在你想要建立远程分支连接的那个分支里面输入这个命令
    * git checkout --track 别名/分支名  自动创建本地分支，分知名为远程分支名并且与远程跟踪分支绑定
    * git checkout -b 分支名 别名/分支名 效果与上面一样     
	* git branch -vv 查看分支是否有远程跟踪分支绑定
## 删除远程分支
    * git push use(别名) --delete (分支名)  删除远程分支
    * git remote prune （别名） --dry-run  列出仍在远程跟踪但是远程分支已经被删除的无用分支
    * git remote rm 别名    清除上面的命令列出来的远程跟踪
## 冲突

   * git本地操作的冲突
        * 典型合并的时候
  	    * git远程协作的时候
        	* push
        		两个人同时推（更改同一个文件）解决办法只能先把远程仓库拉下来 然后再更改那个    文件然后再add commit push  
        	* pull
       	        更改完以后不push 直接pull会报错 远程仓库会覆盖更改的内容建议push 不过push还会出错就是上面那个错误



# 参加开源项目的步骤 （pull request） 
   如果参加某个项目时，但是没有推送权限，这时候可以通过对这个项目进行fork。这会在你的空间中创建一个完全属于你的项目副本，且你对其具有推送权限。通过这个方式，项目的管理者不用忙着添加贡献者，人们可以fork这个项目将修改推送到项目副本上，并通过pull request来将他们的改动进入源版本库
    
   1. 先将源仓库fork到自己的仓库
    2. 然后clone到本地仓库
    3. 更改后提交到自己的远程仓库
    4. 使用pull request提交自己修改的代码给项目管理者参考
    5. 管理人审核 然后merge