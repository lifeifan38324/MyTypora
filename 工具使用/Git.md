# 一.Git的下载与安装

## 1.下载

1. Git官网下载

## 2.安装

1. [看尚硅谷文档](https://www.bilibili.com/video/BV1vy4y1s7k6?p=7)

# 二.文件常用命令

1. 配置用户信息（第一次打开时使用）

    - 用户名：`git config --global user.name`
    - 虚拟邮箱：`git config --global user.email`
    - 查看配置信息路径：`C:\Users\LFF\.gitconfig`
2. 初始化本地库

    - 新建并进入库目录，右键打开`GitBash`
    - 输入命令`git init`，生成一个`.git隐藏文件`
3. 查看**本地库**状态

    - `git status`
        - 红色表示文件未被追踪到，文件存在于**工作区**。
        - 绿色表示文件被追踪到，文件存在于**缓存区**。
    - `git reflog`：查看引用日志信息
        - `HRAD ->`指针，指向当前分区
4. 添加：工作区 ----> 缓存区

    - `git add 文件名`
5. 删除：缓存区文件

    - `git rm --cached 文件名`
6. 提交：缓存区 --> 本地库

    - `git commit -m "日志信息" 文件`
7. 修改代码
    - 修改 --> 添加到缓存区 --> 提交到本地库
8. 版本穿梭
    - `git reset --hard 版本号`
    - 实际是指针控制版本：`head-->master-->first|second|third`
    - 版本切换时，实际是指针`master-->first|second|third`的变化。

# 三.分支操作

理解：初学者可以将分支理解为副本，底层是指针的引用。

1. 查看分支：`git branch -v`

2. 创建分支：`git branch 分支名`

3. 切换分支：`git checkout 分支名`

4. 合并分支：

    - `git merge 分支名` 将分支合并到当前分支上
    - 冲突合并：两个分支在**同一文件的同一位置**有两套完全不同的修改
        - 使用`vim打开文件`，删除多余内容，只剩下最终合并的内容
        - 添加缓存区，提交到本地库（注意这里不需要提交文件名）

5. 分支原理图解

    ![image-20220506183544409](../框架学习/imgs/版本分支原理)

# 四.团队合作：利用代码托管中心

1. 背景资料

    - [尚硅谷团队合作原理视频讲解](https://www.bilibili.com/video/BV1vy4y1s7k6?p=19&spm_id_from=pageDriver)

    - 团队内合作

        图片

    - 跨团队合作

        图片

2. 创建远程库

3. `git`链接`GitHub远程库`

    - 查看远程库：`git remote -v`

    - 创建远程库别名：`git remote add 别名 链接`

    - 将本地代码推送到远程库：`git push 远程库别名 本地分支名`

        - 报错：`remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.即GitHub不支持密码登录`

            解决：申请`token`，登陆时使用`token`代替密码进行登录。
        
    - 添加另一个账户到团队
    
        - 打开主人仓库，`settings`，`Collaborators`，

4. 另一个本地账号：

    - 将远程库代码拉取到本地：`git pull 远程库别名 本地库分支`

    - 克隆远程仓库：`git clone 远程库HTTPS`。注意可以不需要登录账号。

        - 操作：1.拉取代码。2.初始化本地库。3.创建别名。

    - 修改本地状态并提交。

# 五.跨团队协作

1. 另一个团队的账号
    - 登录--->搜索合作文档`“用户名/文件名”` | 输入链接网址--->`fork`
    - 新团队内部修改代码
    - 请求原团队：点击`Pull request`
    - 原团队回应：可以聊天，之后合并代码

# 插：[SSH免密登录](https://www.bilibili.com/video/BV1vy4y1s7k6?p=26&spm_id_from=pageDriver)

1. windows 用户目录找到`.ssh`，删除
2. 在用户目录下右键打开`GitBash`，输入`ssh-keygen -t rsa -C lff38324@qq.com`生成`SSH公钥和私钥`
3. 复制公钥的内容，粘贴到`GitHub，setting，SSH and GPG keys`
4. 成功获取SSH

# 六.`IDEA`集成`Git`

1. 配置忽略文件

    - 在windows的用户目录下，新建`git.ignore`，重要的是后缀

        ```git.ignore
        # Compiled class file
        *.class
        
        # Log file
        *.log
        
        # BlueJ files
        *.ctxt
        
        # Mobile Tools for Java (J2ME)
        .mtj.tmp/
        
        # Package Files #
        *.jar
        *.war
        *.nar
        *.ear
        *.zip
        *.tar.gz
        *.rar
        
        # virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
        hs_err_pid*
        .classpath
        .project
        .settings
        target
        .idea
        *.iml
        ```
        
    - 在`.gitconfig`中添加引用`git.ignore`，注意不要使用反斜线`\`

        ```.gitconfig
        [core]
        	excludesfile = C:/Users/LFF/git.ignore
        ```

2. 打开`idea`工程，`setting`，`Version Control`，`git`，`Path to Git executable`中选择`git.exe`文件

3. 使用`idea`初始化`git本地库`

    - `VCS`，`Import into Version Control`，`Create Git Reposetory`，默认选中项目根目录，OK

4. 添加缓存和提交至本地库。在项目根目录右键git找。

5. 查看版本信息 和 切换版本：左下角`version control`，`log`，右键`Checkout Revision`切换版本

6. 创建分支：

    - 方法一：`Git`，`Repository`，`Branches`，
    - 方法二：点击右下角`Git:master`
    - 查看当前分支看右下角

7. 合并分支

    - 正常合并：首先切换到`master分支`，点击待合并的分支`Merge into Current`
    - 冲突合并：图形操作

# 七.`IDEA`集成`GitHub`

1. 在`IDEA`中使用`Token`登录GitHub账号
2. 创建并首次`push`：VCS，Import into Version Control，Share Project on GitHub
3. 之后的`push`至远程库：
4. 拉取远程库：注意拉取之前先把工作区的文件提交至本地库
5. 克隆`GitHub`的文件：File，Clone Project

# 八.码云`Gitee`

1. 基本操作参考GitHub
2. 迁移GitHub项目到码云：新建仓库是选择导入

# 九.GitLab

1. 需要时再看，[尚硅谷Gitee使用教程](https://www.bilibili.com/video/BV1vy4y1s7k6?p=41&spm_id_from=pageDriver)
