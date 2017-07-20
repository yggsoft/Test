# 创建分支
git checkout -b dev

git branch dev
git checkout dev

git push --set-upstream origin develop

# Merge
git checkout master
git merge dev
git status

git merge --no-ff -m "merge with no-ff" dev

# 删除分支
git branch -d dev
删除远程分支
git push [远程名] [本地分支]:[远程分支] 语法，如果省略 [本地分支]，那就等于是在说“在这里提取空白然后把它变成[远程分支]”。
比如删除远程分支feature1
git push origin :feature1
- [deleted] feature1

# git log
git log --graph --pretty=oneline

# TAG
git tag v1.2.0
默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？
方法是找到历史提交的commit id，然后打上就可以了：它对应的commit id是6224937，敲入命令
git tag v0.9 6224937

git tag -a v0.1 -m "version 0.1 released" 3628164

建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。
如果要推送某个标签到远程，使用命令git push origin <tagname>

# mvn release:clean release:prepare -Dtag=steven -DdevelopmentVersion=3.8-SNAPSHOT -DreleaseVersion=3.9


# 版本规则
Maven主要是这样定义版本规则的：
<主版本>.<次版本>.<增量版本>
比如说1.2.3，主版本是1，次版本是2，增量版本是3。
主版本一般来说代表了项目的重大的架构变更，比如说Maven 1和Maven 2，在架构上已经两样了，将来的Maven 3和Maven 2也会有很大的变化。次版本一般代表了一些功能的增加或变化，但没有架构的变化，比如说Nexus 1.3较之于Nexus 1.2来说，增加了一系列新的或者改进的功能（仓库镜像支持，改进的仓库管理界面等等），但从大的架构上来说，1.3和1.2没什么区别。至于增量版本，一般是一些小的bug fix，不会有重大的功能变化。

一般来说，在我们发布一次重要的版本之后，随之会开发新的版本，比如说，myapp-1.1发布之后，就着手开发myapp-1.2了。由于myapp-1.2有新的主要功能的添加和变化，在发布测试前，它会变得不稳定，而myapp-1.1是一个比较稳定的版本，现在的问题是，我们在myapp-1.1中发现了一些bug（当然在1.2中也存在），为了能够在段时间内修复bug并仍然发布稳定的版本，我们就会用到分支（branch），我们基于1.1开启一个分支1.1.1，在这个分支中修复bug，并快速发布。这既保证了版本的稳定，也能够使bug得到快速修复，也不同停止1.2的开发。只是，每次修复分支1.1.1中的bug后，需要merge代码到1.2（主干）中。

实战分支
目前我们trunk的版本是1.1-SNAPSHOT，其实按照前面解释的版本规则，应该是1.1.0-SNAPSHOT。
现在我们想要发布1.1.0，然后将主干升级为1.2.0-SNAPSHOT，同时开启一个1.1.x的分支，用来修复1.1.0中的bug。
首先，在发布1.1.0之前，我们创建1.1.x分支，运行如下命令：
mvn release:branch -DbranchName=1.1.x -DupdateBranchVersions=true -DupdateWorkingCopyVersions=false
这是maven-release-plugin的branch目标，我们指定branch的名称为1.1.x，表示这里会有版本1.1.1, 1.1.2等等。updateBranchVersions=true的意思是在分支中更新版本，而updateWorkingCopyVersions=false是指不更改当前工作目录（这里是trunk）的版本。
在运行该命令后，我们会遇到这样的提示：
What is the branch version for "Unnamed - org.myorg:myapp:jar:1.1-SNAPSHOT"? (org.myorg:myapp) 1.1-SNAPSHOT: :
——"分支中的版本号是多少？默认为1.1-SNAPSHOT" 这时我们想要的版本是1.1.1-SNAPSHOT，因此输入1.1.1-SNAPSHOT，回车，maven继续执行直至结束。
接着，我们浏览svn仓库，会看到这样的目录：https://192.168.1.100:8443/svn/myapp/branches/1.1.x/，打开其中的POM文件，其版本已经是1.1.1-SNAPSHOT。
分支创建好了，就可以使用release:prepare和release:perform为1.1.0打标签，升级trunk至1.2.0-SNAPSHOT，然后分发1.1.0。
至此，一切OK。

# 三种广泛使用的工作流程
http://www.ruanyifeng.com/blog/2015/12/git-workflow.html
        Git flow
        Github flow
        Gitlab flow
三种工作流程，有一个共同点：都采用"功能驱动式开发"（Feature-driven development，简称FDD）。
它指的是，需求是开发的起点，先有需求再有功能分支（feature branch）或者补丁分支（hotfix branch）。完成开发后，该分支就合并到主分支，然后被删除。

http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013758410364457b9e3d821f4244beb0fd69c61a185ae0000

    master分支是主分支，因此要时刻与远程同步；
    dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
    bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
    feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。


# maven release
release:update-version
修改当前工作区的pom的版本号，包括子模块，及模块之间相互依赖的版本号； mvn --batch-mode release:update-versions -DdevelopmentVersion=1.3.3-SNAPSHOT -DupdateDependencies=true

mvn release:branch -DbranchName=1.1.1 -DreleaseVersion=1.2.1-SNAPSHOT -DupdateBranchVersions=true -DupdateWorkingCopyVersions=false -DupdateVersionsToSnapshot=false -DdryRun=true -DpushChanges=false
        这个打分支的脚本，版本不会修改为 release，还是原来的snapshot
        -DdevelopmentVersion=1.0.3 -DupdateWorkingCopyVersions=true 执行当前工作区新的版本号，如果当前工作区版本号不需要修改，则 DupdateWorkingCopyVersions 设置为false;
        -DdryRun=true 用于在进行验证的时候使用，该参数设置为true，则不会提交到版本控制区，只会在当前版本修改版本号；会生成一个pom.xml.branch的文件；这个是分支的文件；
        -DpushChanges=false 用于git，表示是否往远端推送当前的变更；默认true 会推送；
        -DreleaseVersion=1.0.1-SNAPSHOT -DupdateBranchVersions=true 用于指定新的分支的版本号；新分支的版本号如果要修改，则：updateBranchVersions 必须设置为true;
        -DupdateVersionsToSnapshot=true 是否将分支版本修改为 Snapshot

mvn release:clean release:prepare -DupdateWorkingCopyVersions=false -DreleaseVersion=1.0.1-RELEASE -Dtag=busi-1.0.1-RELEASE -DignoreSnapshots=true
    -DupdateWorkingCopyVersions=false 设置为false 表示不修改当前工作区的版本号；如果设置为 true，则需要增加一个参数：-DdevelopmentVersion=1.0-SNAPSHOT表示当前工作区新的版本号； -DreleaseVersion=1.0.1-RELEASE 表示打的tag的版本号； -Dtag=busi-1.0.1-RELEASE -DtagNameFormat=@{project.artifactId}-@{project.version} 表示tag的名字，默认的tagNameFormat就是@{project.artifactId}-@{project.version}，所以没有配置 tag ，也会有tag出来，而且tag的名字就是根据tagNameFormat出来的格式；


https://my.oschina.net/u/140938/blog/666695

# Git Fetch
git fetch 有四种基本用法
1. git fetch            →→ 这将更新git remote 中所有的远程repo 所包含分支的最新commit-id, 将其记录到.git/FETCH_HEAD文件中
2. git fetch remote_repo         →→ 这将更新名称为remote_repo 的远程repo上的所有branch的最新commit-id，将其记录。 
3. git fetch remote_repo remote_branch_name        →→ 这将这将更新名称为remote_repo 的远程repo上的分支： remote_branch_name
4. git fetch remote_repo remote_branch_name:local_branch_name       →→ 这将这将更新名称为remote_repo 的远程repo上的分支： remote_branch_name ，并在本地创建local_branch_name 本地分支保存远端分支的所有数据。
FETCH_HEAD： 是一个版本链接，记录在本地的一个文件中，指向着目前已经从远程仓库取下来的分支的末端版本。

# Git Remote
本地的repo仓库要与远程的repo配合完成版本对应必须要有 git remote子命令，通过git remote add来添加当前本地长度的远程repo， 有了这个动作本地的repo就知道了当遇到git push 的时候应该往哪里提交代码。


http://www.oschina.net/translate/10-tips-git-next-level
http://pinkyjie.com/categories/%E5%B7%A5%E5%85%B7%E7%9B%B8%E5%85%B3/
