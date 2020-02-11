# 一些语法

## markdown语法

1. [百度](www.baidu.com)

2. ![内嵌图片alt text](文本url)

3. <muyouoi@outlook.com>

        缩进四个空格可以写代码块
        nice啊
        不过的区别是不支持代码高亮

4. 末尾加两个空格再回车换行  
   你看

5. 三个***是水平分割线
    ***

6. ~~两边加上两个~等于删除线~~

7. >连续引用
   >
   >不会断掉

8. 这是表格
    header 1 | header 2 | header 3
    -- | :--: | --:
    left | center | right

 ***

## 命令行语法

删除文件

    sudo rm -rf 文件夹名 // 删除文件夹以及文件夹内文件
    rm 文件名1 文件名2   // 同时删除多个文件

创建文件

    mkdir 目录名  // 新建一个文件夹
    touch 文件名  // 新建一个空文件

拷贝文件

    cp 文件名 目标路径 // 拷贝一个文件到目标路径

移动文件

    mv 文件名 目标路径 // 移动一个文件到目标路径

文件路径

    pwd    // 显示当前文件路径
    cd     // 跳向文件夹
    cd ~   // 切换到当前用户的主目录
    cd ..  // 切换至上级目录

查看文件

    cat 文件名  // 查看文件内容
    ls         // 显示路径下所有文件
    ls-l       // 以列表形式显示文件详细信息

***

## git 语法

    git init         // 初始化  
    git init newrepo // 指定newrepo目录作为git仓库并初始化

    git clone <repo>             // 克隆远程仓库repo
    git clone <repo> <directory> // 克隆远程仓库repo到directory
    git clone git.git newgit     // 自定义仓库名称

    git add 文件名 // 将文件添加到暂存区
    git status    // 查看修改状态
    git diff      // 显示已写入缓存 与 已修改但尚未写入缓存的改动的不同之处
    git commit -m "描述信息"  // 将暂存区文件写入仓库
    git commit -am "描述信息" // 直接从工作区到仓库

    git reset HEAD 文件名 // 撤销上传到暂存区的修改，使其回到工作区

    git rm <file>    // 删除文件
    git rm -f <file> // 如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f
    git rm --cached <file> 
    // 如果把文件从暂存区域移除，但仍然希望保留在当前工作目录中，换句话说，仅是从跟踪清单中删除
    git rm –r * 
    // 可以递归删除，即如果*是一个目录作为参数，则会递归删除整个目录中的所有子目录和文件

    git mv README README.md 
    // git mv 命令用于移动或重命名一个文件、目录、软连接
***

## vsCode 快捷键
