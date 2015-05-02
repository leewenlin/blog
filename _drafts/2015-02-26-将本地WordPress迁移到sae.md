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
   ```c
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
   ```
2. 将wp的uploads目录替换为sae的Storage
   - 在storage中新建domain
   - 项目根目录新建sae.php文件
   - 修改functions.php文件
