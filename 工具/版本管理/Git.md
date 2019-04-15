# Git
## 1. Git 配置文件
Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量,这些变量存放在是三个不同的位置:
    - etc/gitconfig : 设置系统每一个用户及其仓库的通用配置.
    - ~/.gitconfig : 针对当前用户,全局配置
    - 当前本地仓库内的 .git/config 文件 : 针对当前仓库

- 查看三个配置文件内容
    - 下面三个文件,==*优先级依次增高*==
    - 查看 etc/gitconfig
    
        ```shell
        git config --system -l
        ```
 
    - 查看 ~/.gitconfig
        
        ```shell
        git config --global -l
        ```
    
    - 查看当前使用仓库的 config
        
        ```shell
        git config --local -l
        ```
    
- 设置全局的配置. 安装完 Git 第一件事就是要配置 global 配置,每一次 Git 提交都会用到这些信息


    ```shell
    git config --global user.name "设置用户名"
    git config --global user.email "设置邮箱" 
    ```
    
    - 如果设置了 global 选项,那么在系统上做任何事,都是通过配置的 global 用户和邮箱进行操作,==**如果想针对特性项目使用不同的用户名称和邮箱地址,则可以在那个项目目录下执行没有 --global 选项的命令来配置.**==

- ==*查看最终生效的配置*==

    ```shell
    git config --list
    ```
    
    
## 2. 取得项目的 Git 仓库
- 有两种方式 : 
    - 第一种 : 在现存的目录下,通过导入所有文件来常见新的 Git 仓库 (==*git init*==)
    - 第二种 : 从已有的 Git 仓库克隆出一个新的镜像仓库(==*git clone*==)

### 2.1 git init
- 对想对现有的某个项目开始使用 Git 进行管理,只要在此项目所在的目录执行以下命令即可 : 

    ```shell
    git init 
    ```

- 初始化后,在当前目录下会生成一个 .git 文件夹,所有 Git 需要的数据和资源都在这个文件夹内.
- 此时只是按照目录现有的结构初始化好了里面的文件和目录,还没有开始管理项目中的任何一个文件.
- 要将当前目录下的几个文件纳入版本管理,需要使用 git add 进行添加


### 2.2 git clone 