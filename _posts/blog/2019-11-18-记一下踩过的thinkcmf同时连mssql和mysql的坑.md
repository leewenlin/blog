---
layout: post
title: 记一下踩过的thinkcmf同时连mssql和mysql的坑
modified:
categories: 
excerpt: 
tags: [php,thinkcmf,mssql]
comments: true
share: true
---

之前踩过thinkcmf同时连mssql和mysql的坑，搜解决办法的时候，也都是问的人多，没人给出方案，正好有空，就写写完整的方案了_(:з」∠)_

### 环境
我是win下的，用的wamp+php7.1+mssql08+mysql+thinkcmf5（其中mssql是Chinese_PRC_BIN排序的，会触发thinkphp的一个坑，后面会说），其实Linux下也一样搞，区别并不大

### 下载&&安装
1. 安装ODBC Driver，[下载链接](https://www.microsoft.com/zh-CN/download/details.aspx?id=36434)
2. 下载Microsoft Drivers for PHP for SQL Server，[下载链接](https://www.microsoft.com/en-us/download/details.aspx?id=20098)，找到对应版本的扩展放到ext文件夹中（我用到的是：php_sqlsrv_71_ts_x64.dll和php_pdo_sqlsrv_71_ts_x64.dll）
3. 并且在php.ini中加入这两个扩展，如果跟我一样用的是wamp，实际上需要修改的文件是phpForApache.ini
```
extension=php_pdo_sqlsrv_71_ts_x64.dll 
extension=php_sqlsrv_71_ts_x64.dll
```

### 修改驱动
Chinese_PRC_BIN排序会区分大小写，而thinkphp的数据库连接驱动获取表结构的时候用的是小写，如果不修改驱动，连接的时候会报错“对象名 'information_schema.tables' 无效”。

修改的文件为\vendor\thinkphp\library\think\db\connector\Sqlsrv.php，方法就是把information_schema、tables、columns等等相关字段全部改成大写，偷懒的话可以直接用下面我改的。

```php
<?php

namespace think\db\connector;

use PDO;
use think\db\Connection;
use think\db\Query;

/**
 * Sqlsrv数据库驱动
 */
class Sqlsrv extends Connection
{
    // PDO连接参数
    protected $params = [
        PDO::ATTR_CASE              => PDO::CASE_NATURAL,
        PDO::ATTR_ERRMODE           => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_ORACLE_NULLS      => PDO::NULL_NATURAL,
        PDO::ATTR_STRINGIFY_FETCHES => false,
    ];

    protected $builder = '\\think\\db\\builder\\Sqlsrv';

    /**
     * 解析pdo连接的dsn信息
     * @access protected
     * @param  array $config 连接信息
     * @return string
     */
    protected function parseDsn($config)
    {
        $dsn = 'sqlsrv:Database=' . $config['database'] . ';Server=' . $config['hostname'];

        if (!empty($config['hostport'])) {
            $dsn .= ',' . $config['hostport'];
        }

        return $dsn;
    }

    /**
     * 取得数据表的字段信息
     * @access public
     * @param  string $tableName
     * @return array
     */
    public function getFields($tableName)
    {
        list($tableName) = explode(' ', $tableName);
        $tableNames      = explode('.', $tableName);
        $tableName       = isset($tableNames[1]) ? $tableNames[1] : $tableNames[0];

        $sql = "SELECT   column_name,   data_type,   column_default,   is_nullable
        FROM    INFORMATION_SCHEMA.TABLES AS t
        JOIN    INFORMATION_SCHEMA.COLUMNS AS c
        ON  t.table_catalog = c.table_catalog
        AND t.table_schema  = c.table_schema
        AND t.table_name    = c.table_name
        WHERE   t.table_name = '$tableName'";

        $pdo    = $this->query($sql, [], false, true);
        $result = $pdo->fetchAll(PDO::FETCH_ASSOC);
        $info   = [];

        if ($result) {
            foreach ($result as $key => $val) {
                $val                       = array_change_key_case($val);
                $info[$val['column_name']] = [
                    'name'    => $val['column_name'],
                    'type'    => $val['data_type'],
                    'notnull' => (bool) ('' === $val['is_nullable']), // not null is empty, null is yes
                    'default' => $val['column_default'],
                    'primary' => false,
                    'autoinc' => false,
                ];
            }
        }

        $sql = "SELECT column_name FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE table_name='$tableName'";

        // 调试开始
        $this->debug(true);

        $pdo = $this->linkID->query($sql);

        // 调试结束
        $this->debug(false, $sql);

        $result = $pdo->fetch(PDO::FETCH_ASSOC);

        if ($result) {
            $info[$result['column_name']]['primary'] = true;
        }

        return $this->fieldCase($info);
    }

    /**
     * 取得数据表的字段信息
     * @access public
     * @param  string $dbName
     * @return array
     */
    public function getTables($dbName = '')
    {
        $sql = "SELECT TABLE_NAME
            FROM INFORMATION_SCHEMA.TABLES
            WHERE TABLE_TYPE = 'BASE TABLE'
            ";

        $pdo    = $this->query($sql, [], false, true);
        $result = $pdo->fetchAll(PDO::FETCH_ASSOC);
        $info   = [];

        foreach ($result as $key => $val) {
            $info[$key] = current($val);
        }

        return $info;
    }

    /**
     * 得到某个列的数组
     * @access public
     * @param  Query     $query 查询对象
     * @param  string    $field 字段名 多个字段用逗号分隔
     * @param  string    $key   索引
     * @return array
     */
    public function column(Query $query, $field, $key = '')
    {
        $options = $query->getOptions();

        if (empty($options['fetch_sql']) && !empty($options['cache'])) {
            // 判断查询缓存
            $cache = $options['cache'];

            $guid = is_string($cache['key']) ? $cache['key'] : $this->getCacheKey($query, $field);

            $result = Container::get('cache')->get($guid);

            if (false !== $result) {
                return $result;
            }
        }

        if (isset($options['field'])) {
            $query->removeOption('field');
        }

        if (is_null($field)) {
            $field = '*';
        } elseif ($key && '*' != $field) {
            $field = $key . ',' . $field;
        }

        if (is_string($field)) {
            $field = array_map('trim', explode(',', $field));
        }

        $query->setOption('field', $field);

        // 生成查询SQL
        $sql = $this->builder->select($query);

        $bind = $query->getBind();

        if (!empty($options['fetch_sql'])) {
            // 获取实际执行的SQL语句
            return $this->getRealSql($sql, $bind);
        }

        // 执行查询操作
        $pdo = $this->query($sql, $bind, $options['master'], true);

        if (1 == $pdo->columnCount()) {
            $result = $pdo->fetchAll(PDO::FETCH_COLUMN);
        } else {
            $resultSet = $pdo->fetchAll(PDO::FETCH_ASSOC);

            if ('*' == $field && $key) {
                $result = array_column($resultSet, null, $key);
            } elseif ($resultSet) {
                $fields = array_keys($resultSet[0]);
                $count  = count($fields);
                $key1   = array_shift($fields);
                $key2   = $fields ? array_shift($fields) : '';
                $key    = $key ?: $key1;

                if (strpos($key, '.')) {
                    list($alias, $key) = explode('.', $key);
                }

                if (3 == $count) {
                    $column = $key2;
                } elseif ($count < 3) {
                    $column = $key1;
                } else {
                    $column = null;
                }

                $result = array_column($resultSet, $column, $key);
            } else {
                $result = [];
            }
        }

        if (isset($cache) && isset($guid)) {
            // 缓存数据
            $this->cacheData($guid, $result, $cache);
        }

        return $result;
    }

    /**
     * SQL性能分析
     * @access protected
     * @param  string $sql
     * @return array
     */
    protected function getExplain($sql)
    {
        return [];
    }
}
```

### 配置多个数据库
thinkcmf配置多个数据库其实相当方便。

如果是不同的app用不同库的话，直接在\app\xxxx\config\database.php里配置就好了。

如果是不同的app用不同库,但是权限管理想用公共的，可以在database.php里加个db_con2，然后app的basemodel里加上```protected $connection = 'db_con2';```就行

还有个需要注意的问题就是表名了，mssql的表名通常是大写，按照thinkphp默认的处理，会自动加下划线断开并转小写，例如PANDA会变成p_a_n_d_a，所以模型里面需要加上表名```protected $table = 'dbo.XXXXX';```

### 貌似就这么多需要注意的地方了，这应该是目前为止网上最全面的一篇了
