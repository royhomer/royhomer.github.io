---
layout: post
title: "搭建一个github博客基于Octopress"
date: 2014-07-03 17:10:04 +0800
comments: true
categories: Octopress博客搭建
---

<!--more-->
### 准备工作

确保安装了git和ruby1.9.3,通过brew安装rbenv然后用rbenv安装ruby。

##### 1.安装 rbenv
	
	brew update
	brew install rbenv
	brew install ruby-build
	rbenv rehash
	
##### 2.安装 octopress
	
从GitHub上clone源码：

	git clone git://github.com/imathis/octopress.git octopress
	cd octopress	
	
接着安装依赖项(dependencies)：

	gem install bundler
	rbenv rehash    
	bundle install
	rake install
	
##### 3.部署博客

	rake setup_github_pages
	
##### 4.输入你的github地址
	
	git@github.com:your_username/your_username.github.com
	
##### 5.下来就可以生成静态网页，并将其部署到github上了，只需如下两句：


	rake generate    #生成静态网页
	rake deploy   #发布网页

等待几分钟后，网页就已经部署好了，你可以访问你的域名查看自己的博客了。然后可以将源码添加到github中进行管理，你仓库中有两个分支，master分支是静态网页，source分支则是你的生成网页的源码，这样在另一台电脑上你可以clone你的仓库，很容易的进行你的博客编写。

	git add .
	git commit -m 'init'
	git push origin source

现在就可以进行文章的编写了，编写文章使用markdown语法，十分便捷，语法简单，可参见markdown语法说明。输入如下命令：

##### 6.写文章
	rake new_post['title']

在~/blog/source/_posts下回生成一个名为YYYY-MM-DD-title.markdown的文件，在这个文件中编写你的文章即可。编写完成后依然按上边发布网页的方法发布即可：

	rake generate    #生成静态网页
	rake deploy   #发布网页

或者也可以生成静态网页后，在本机进行预览

	rake preview

然后可以通过```http://localhost:4000```访问，状态满意即可发布网页，以上几步同样适用于更改网页布局、样式等，重新发布网页。
	
	
### octopress博客的个性化配置
###### 添加文章分类（category)

1、增加category_list插件
首先，保存以下内容到```plugins/category_list_tag.rb``中（如果文件不存在就新创建一个）：
	
	module Jekyll
    	class CategoryListTag < Liquid::Tag
        	def render(context)
            	html = ""
            	categories = context.registers[:site].categories.keys
            	categories.sort.each do |category|
                	posts_in_category = context.registers[:site].categories[category].size
                	category_dir = context.registers[:site].config['category_dir']
                	category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase)
                	# html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n"
               	 html << "<li class='category'><a href='/blog/categories/#{category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').to_url.downcase}/'>#{category} (#{posts_in_category})</a></li>\n"
            	end
            	html
       	 	end
    	end
	end

	Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
	


2、再增加aside，复制以下代码到source/_includes/asides/category_list.html（如果没有就新建）:

	<section>
 		<h1>文章分类</h1>
 		<ul id="categories">
  		{% category_list %}
 		</ul>
	</section>
	
	
3、更改```_config.yml```文件，让侧边栏链接到刚才新增加的```source/_includes/asides/category_list.html```文件：


	default_asides: [asides/recent_posts.html, asides/category_list.html, asides/github.html, asides/delicious.html, asides/pinboard.html, asides/googleplus.html]


完成以上步骤后，重新部署就能看到博客的右侧边栏增加了category列表了。




###### 自定义Navigation

默认的导航栏只有Blog、Archives两项，很难满足大家的要求。下面以增加about界面为例说明如何在导航栏上增加内容。

1、首先编辑文件```/source/_includes/custom/navigation.html```，仿照Blog和Archives的写法增加一行About：

	<ul class="main-navigation">
  		<li><a href="{{ root_url }}/">Blog</a></li>
  		<li><a href="{{ root_url }}/blog/archives">Archives</a></li>
  		<li><a href="{{ root_url }}/about">About</a></li>
	</ul>
	

2、然后使用命令：

	rake new_page['about']

创建一个页面，保存路径为```source\about\index.markdown```

编辑index.markdown文件成自己想要的样式，然后重新部署，就能看到导航栏上新增了About项目。


#### 参考文章：
[象写程序一样写博客：搭建基于github的博客](http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/)

[Octopress博客搭建及目录结构](http://812lcl.com/blog/2013/10/25/octopressbo-ke-da-jian-ji-mu-lu-jie-gou/)

[Octopress主题样式修改](http://812lcl.com/blog/2013/10/27/octopresszhu-ti-yang-shi-xiu-gai/)

[自定义你的Octopress博客](http://foggry.com/blog/2014/04/28/custom-your-octopress-blog/)