---
layout: post
title: 将本地WordPress迁移至sae
modified:
categories: 
excerpt: 
tags: [WordPress, SAE]
comments: true
share: true
---

1. 用phpmyadmin导出本地的数据库,导入到sae的mysql中。并修改wp-config.php
   {% highlight php %}
define('DB_NAME', SAE_MYSQL_DB);
define('DB_USER', SAE_MYSQL_USER);
define('DB_PASSWORD', SAE_MYSQL_PASS);
define('DB_HOST', SAE_MYSQL_HOST_M.':'.SAE_MYSQL_PORT);
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');
define('WP_USE_MULTIPLE_DB', true);
$db_list = array(
'write'=> array(
array(
'db_host' => SAE_MYSQL_HOST_M.':'.SAE_MYSQL_PORT,
'db_user'=> SAE_MYSQL_USER,
'db_password'=> SAE_MYSQL_PASS,
'db_name'=> SAE_MYSQL_DB,
'db_charset'=> 'utf8'
)
),
'read'=> array(
array(
'db_host' => SAE_MYSQL_HOST_S.':'.SAE_MYSQL_PORT,
'db_user'=> SAE_MYSQL_USER,
'db_password'=> SAE_MYSQL_PASS,
'db_name'=> SAE_MYSQL_DB,
'db_charset'=> 'utf8'
)
),
);
$global_db_list = $db_list['write'];
   {% endhighlight %}

2. 将wp的uploads目录替换为sae的Storage
   * 在storage中新建domain
   * 项目根目录新建sae.php文件
   {% highlight php %}
/* 在SAE的Storage中新建的Domain名*/
define('SAE_STORAGE','Domain');
/* 设置文件上传的路径和文件路径的URL，不要更改 */
define('SAE_DIR', 'saestor://'.SAE_STORAGE.'/uploads');
define('SAE_URL', 'http://'.$_SERVER['HTTP_APPNAME'].'-'.SAE_STORAGE.'.stor.sinaapp.com/uploads');
   {% endhighlight %}
   * 修改functions.php文件
     首行插入
     {% highlight php %}
include( ABSPATH . '/sae.php' );
     {% endhighlight %}
     wp_mkdir_p( $target ) 中修改如下
     {% highlight php %}
function wp_mkdir_p( $target ) {
//注释
//  $wrapper = null;
//
//  // Strip the protocol.
//  if( wp_is_stream( $target ) ) {
//      list( $wrapper, $target ) = explode( '://', $target, 2 );
//  }
//
//  // From php.net/mkdir user contributed notes.
//  $target = str_replace( '//', '/', $target );
//
//  // Put the wrapper back on the target.
//  if( $wrapper !== null ) {
//      $target = $wrapper . '://' . $target;
//  }
//新增
        return true;
    $target = str_replace( '//', '/', $target );
     {% endhighlight %}
     {% endhighlight %}$basedir = $dir;{% endhighlight %}的上面添加
     {% highlight php %}
$dir = SAE_DIR;$url = SAE_URL;
     {% endhighlight %}
     {% endhighlight %}/** * Send a HTTP header to limit rendering of pages to same origin iframes.{% endhighlight %}的上面添加
     {% highlight php %}
if ( !function_exists('utf8_encode') ) {
function utf8_encode($str) {
$encoding_in = mb_detect_encoding($str);
return mb_convert_encoding($str, 'UTF-8', $encoding_in);
}
}
     {% endhighlight %}
   * 修改wp-admin/includes/file.php，注释掉如下代码
    {% highlight php %}
// Set correct file permissions
//$stat = stat( dirname( $new_file ));
//$perms = $stat['mode'] & 0000666;
//@ chmod( $new_file, $perms );
    {% endhighlight %}