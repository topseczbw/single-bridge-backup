hexo通过将markdown文件生成静态html，并借助github为服务器，快速搭建属于自己的博客。

#### 开始搭建

1. ##### 安装依赖

    在node、git前提下，全局安装hexo。

    ```shell
     sudo npm i hexo -g
    ```

2. ##### 初始化仓库

   执行下述命令，然后通过  localhost:4000  查看。

    ```shell
    mkdir blog && cd blog
    hexo init	#初始化hexo仓库，默认安装依赖，如果没有，需要手动 npm install
    hexo g		#生成缓存和静态文件，可以理解为编译markdown为html
    hexo s		#开发环境调试博客
    ```

3. ##### 关联个人github

   注册并登陆GitHub账号后，新建public仓库，名称必须是  {% hl_text danger %}topseczbw.github.io（前缀是用户名）{% endhl_text %}  。

   修改本地仓库_config.yml文件，打开后到文档最后部分，将deploy配置如下：

    ```yml
    deploy:
      type: git
      repository: https://github.com/topseczbw/topseczbw.github.io.git
      branch: master
    ```

4. ##### 安装hexo-deployer-git

   git与hexo仓库关联所依赖的推送工具，重新编译并上传github。

   若未关联GitHub，执行`hexo d`时会提示输入GitHub账号用户名和密码。

   ```shell
    npm i hexo-deployer-git -S	 #安装推送工具
    hexo g && hexo d			 #重新部署到服务器
   ```

#### 添加ssh keys到github

1. ##### 为什么设置？

   添加ssh key后不需要每次更新博客再输入用户名和密码。

2. ##### 如何设置？

   首先检查本地是否包含ssh keys。如果存在则直接将ssh key添加到GitHub之中，否则新生成ssh key。

   执行下述命令生成新的ssh key，将  `your_email@exampl`  改成自己注册的GitHub邮箱地址。默认会在  `~/.ssh/id_rsa.pub`  中生成  `id_rsa`  和  `id_rsa.pub`  文件

     ```shell
      ssh-keygen -t rsa -C "your_email@exampl"        
     ```

   Mac下利用  `open ~/.ssh`  打开文件夹，打开id_rsa.pub文件，里面的信息即为ssh key，将此信息复制到GitHub的Add ssh key  `路径GitHub->Setting->SSH keys->add SSH key`  中即可。Title里填写任意标题，将复制的内容粘贴到key中，点击Add key完成添加

   此时本地博客内容便已关联到GitHub之中，本地博客改变之后，通过  `hexo g`  和  `hexo d`  便可更新到GitHub之中，通过 https://topseczbw.github.io/ 访问便可看到更新内容

  

#### 搭建博客相关资源

1. [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
2. [Codeing Page](https://coding.net/pages)
3. [Hexo 从 GitHub 到阿里云](https://zhuanlan.zhihu.com/p/58654392)
4. [百度站点管理](https://ziyuan.baidu.com/site/index)
5. [如何配置 SSH 公钥访问 git 仓库？](https://coding.net/help/doc/git/ssh-key.html)
6. [Coding Pages 托管](https://blog.csdn.net/qq_36667170/article/details/79318665)
7. [Hexo博客提交百度和Google收录](https://www.jianshu.com/p/f8ec422ebd52)
8. [腾讯云开发平台 = coding ≈ github](https://dev.tencent.com/u/singlebridge/p/singlebridge/git/pages/settings)