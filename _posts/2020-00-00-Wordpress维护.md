---
title: Wordpress
author: Even
date: 2020-07-23 14:23:00 +0800
categories: [CS, Web]
tags: [wordpress, web]
toc: false
---

[用nginx+WordPress搭建个人博客全流程](https://segmentfault.com/a/1190000018964702)

 

**维护**

“建立数据库连接时出错”：重启数据库
`systemctl restart mariadb`



## 为主题增加分页功能

`wordpress/wp-content/themes/[theme name]`目录下：

+ `functions.php`增加分页代码：

  ```
  function wp_pagenavi(){
  	global $wp_query;
   
  	$big = 999999999; // 需要一个不太可能的整数
   
  	$pagination_links = paginate_links( array(
  		'base' => str_replace( $big, '%#%', esc_url( get_pagenum_link( $big ) ) ),
  		'format' => '?paged=%#%',
  		'current' => max( 1, get_query_var('paged') ),
  		'total' => $wp_query->max_num_pages
  	) );
   
  echo $pagination_links;
  }
  ```

+ 在需要分页的地方添加：

  ```php
  wp_pagenavi();
  ```

  需要分页的地方也是在当前主题文件夹下的`index.php` `archive.php` `search.php` `home.php`四个文件里，把`the_posts_navigation();`换成pagenavi函数就可以了。

