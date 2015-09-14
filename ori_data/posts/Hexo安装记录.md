title: Hexo安装记录
---
## 经历
安装Hexo的过程中走了不弯路，所以就自己写个记录，以示他人

## 环境
### git:       
[GitHub Windows](https://desktop.github.com/),你懂的
    
[git](https://git-scm.com/),这个没被墙
### Node.JS:
[Node.JS](http://nodejs.org/)     
####### 注意 安装完成后添加Path环境变量，使```  npm  ```命令生效
### 安装Hexo

	 1 npm install -g hexo-cli 
	 2 npm install hexo --save 


[https://hexo.io/zh-cn/docs/index.html#安装](https://hexo.io/zh-cn/docs/index.html#安装)

## Hexo初始化配置
### 创建Hexo文件夹
安装完成后，根据自己喜好建立目录（如``` D:\nodejspojo\hexo ```），进入Git Bash切换到该路径下``` D:\nodejspojo\hexo```执行以下指令
	1 hexo init
### 安装Hexo插件
	npm install
这样装会漏掉部分插件,如:	    
hexo-deployer-git

	npm install hexo-deployer-git --save

详细插件的安装:

	npm install hexo-generator-index --save
	npm install hexo-generator-archive --save
	npm install hexo-generator-category --save
	npm install hexo-generator-tag --save
	npm install hexo-server --save
	npm install hexo-deployer-git --save
	npm install hexo-deployer-heroku --save
	npm install hexo-deployer-rsync --save
	npm install hexo-deployer-openshift --save
	npm install hexo-renderer-marked@0.2 --save
	npm install hexo-renderer-stylus@0.2 --save
	npm install hexo-generator-feed@1 --save
	npm install hexo-generator-sitemap@1 --save


### 本地查看效果
继续执行以下命令，成功后可登录``` localhost:4000 ```查看效果   

	hexo server
### Hexo简写命令
	hexo n #new
	hexo g #generate
	hexo s #server

## 部署静态网页到GitHub
### 注册设置GitHub
-登录[GitHub](https://github.com/)，注册自定义用户名如wsgzao    
-在主页右上角创建``` New repository```，name必须和用户名一致如```wsgzao.github.io```    
-现在创建并没有静态主页。<s>首次创建耐心等待10分钟左右审核，之后即可访问静态主页如```http://wsgzao.github.io```</s>

### 同步内容至GitHub
部署到Github前需要配置```_config.yml```文件    
#### Git
安装 hexo-deployer-git。    
	npm install hexo-deployer-git --save
修改配置。

	deploy:
	  type: git
	  repo: <repository url>
	  branch: [branch]
	  message: [message]


	|参数       |描述                                                          |
	|-----------|------------------------------------------------------------:|     
	|repo       |库（Repository）地址                                            |
	|branch     |分支名称。如果您使用的是 GitHub 或 GitCafe 的话，程序会尝试自动检测 |
	|message    |自定提交信息                                                   |

[https://hexo.io/zh-cn/docs/deployment.html](https://hexo.io/zh-cn/docs/deployment.html)

之后在``` Git```命令框(Git Bash)下，执行下列指令即可完成部署，注意部署会覆盖掉你之前在版本库中存放的文件。

	1 hexo clean
	2 hexo generate
	3 hexo deploy

### 注意
 - 只要执行了``` hexo deploy ```,那么它就会将```hexo generate```所产生的public文件夹内容部署上```_config.yml```文件设置的Git地址了,成功的你就可以到你的主页上看到了。而当前文件夹的内容是不用提交上Git,否则你每次提交都只是当前文件夹的内容,不要问我怎样知道的···

## 最后
### Md语法与编辑器
[editor.md](https://pandao.github.io/editor.md/)

## 参考文章: 
[使用hexo搭建博客](http://yangjian.me/workspace/building-blog-with-hexo/)    
[使用GitHub和Hexo搭建免费静态Blog](http://wsgzao.github.io/post/hexo-guide/)  
[将原始的 .md 文件纳入 hexo 的版本管理](http://baurine.github.io/2015/05/10/hexo_git/)