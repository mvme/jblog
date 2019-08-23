### 我的博客

感谢 [suyan](https://github.com/suyan/suyan.github.io)
主题来源：[http://yansu.org](http://yansu.org)

### 安装说明

1. fork库到自己的github
2. 修改名字为：`username.github.io`
3. clone库到本地，参考`_posts`中的目录结构自己创建适合自己的文章目录结构
4. 修改CNAME，或者删掉这个文件，使用默认域名
5. 修改`_config.yml`配置项
6. It's done!

### 分支说明

- 三栏布局（master分支，基于[3-Jekyll](https://github.com/P233/3-Jekyll)）
- 三栏布局 (bootstrap-based分支，基于Bootstrap)
- 单栏布局（first-ui分支，基于Bootstrap）



### 基于分支gh-pages修改说明

1. `_config.yml`
   1. baseurl改为自己的repo名字，e.g. /jblog/
   2. avatar个人头像图片
   3. favicon网站小图标，如果来源于网络，尽量使用https
   4. disqus评论已关闭
2. 页面中所涉及到的http都改成了https
3. _includes中sidebar.html的导航链接修改为{{ post.url | prepend: site.baseurl }}
4. _sass新增了table的样式，并修复了10-posttoc.sass中#post-toc样式显示不全的问题
5. css中把修改的table样式加入到博客中去