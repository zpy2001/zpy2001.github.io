---
title: PHP - 学习笔记
description: Notes about PHP
slug: language-php
date: 2022-07-05 00:00:00+0000
categories:
    - Language
tags:
    - study
    - Database
    - Language
    - PHP
    - CS
weight: 1
---

# php

## php + sql

### 连接数据库

```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";

// mysqli创建连接
$conn = new mysqli($servername, $username, $password);
$conn = mysqli_connect($servername, $username, $password);
 
// 检测连接
if ($conn->connect_error) {
    die("连接失败: " . $conn->connect_error);
} 
echo "连接成功";

// pdo
try {
    $conn = new PDO("mysql:host=$servername;", $username, $password);
    echo "连接成功"; 
}
catch(PDOException $e)
{
    echo $e->getMessage();
}


// 关闭连接
$conn->close();
mysqli_close($conn);
// pdo
$conn = null;
?>
```

### sql语句直接执行

```php
// 创建数据库
$sql = "CREATE DATABASE myDB";
if ($conn->query($sql) === TRUE) {
    echo "数据库创建成功";
} else {
    echo "Error creating database: " . $conn->error;
}

try {
    $sql = "CREATE DATABASE myDBPDO";
    // 使用 exec() ，因为没有结果返回
    $conn->exec($sql);
    echo "数据库创建成功<br>";
}
catch(PDOException $e)
{
    echo $sql . "<br>" . $e->getMessage();
}
```

以后操作可以直接修改

### 预处理后执行

```php
//预处理及绑定
$stmt = $conn->prepare("INSERT INTO MyGuests (firstname, lastname, email) VALUES (?, ?, ?)");
$stmt->bind_param("sss", $firstname, $lastname, $email);
// 设置参数并执行
$firstname = "John";
$lastname = "Doe";
$email = "john@example.com";
$stmt->execute();
```

### 查询

获取结果：

```php
// mysqli
$result = $conn->query($sql);
$result = mysqli_query($conn, $sql);
// pdo
$stmt = $conn->prepare("SELECT id, firstname, lastname FROM MyGuests");
$stmt->execute();
// set the resulting array to associative
$result = $stmt->setFetchMode(PDO::FETCH_ASSOC);
foreach(new TableRows(new RecursiveArrayIterator($stmt->fetchAll())) as $k=>$v) {
    echo $v;
}
```

### 获取插入行id

```php
// mysqli
$last_id = $conn->insert_id;
$last_id = mysqli_insert_id($conn);
// pdo
$last_id = $conn->lastInsertId();
```

### 连接字符串

```php
$sql = "";
$sql .= "";
```

### pdo错误处理

```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";
$dbname = "myDBPDO";

try {
  $conn = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
  // set the PDO error mode to exception
  $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

  // begin the transaction
  $conn->beginTransaction();
  // our SQL statements
  $conn->exec("INSERT INTO MyGuests (firstname, lastname, email)
  VALUES ('John', 'Doe', 'john@example.com')");
  $conn->exec("INSERT INTO MyGuests (firstname, lastname, email)
  VALUES ('Mary', 'Moe', 'mary@example.com')");
  $conn->exec("INSERT INTO MyGuests (firstname, lastname, email)
  VALUES ('Julie', 'Dooley', 'julie@example.com')");

  // commit the transaction
  $conn->commit();
  echo "New records created successfully";
} catch(PDOException $e) {
  // roll back the transaction if something failed
  $conn->rollback();
  echo "Error: " . $e->getMessage();
}

$conn = null;
?>
```

MySQL Functions

- mysql_affected_rows — Get number of affected rows in previous MySQL operation
- mysql_client_encoding — Returns the name of the character set
- mysql_close — Close MySQL connection
- mysql_connect — Open a connection to a MySQL Server
- mysql_create_db — Create a MySQL database
- mysql_data_seek — Move internal result pointer
- mysql_db_name — Retrieves database name from the call to mysql_list_dbs
- mysql_db_query — Selects a database and executes a query on it
- mysql_drop_db — Drop (delete) a MySQL database
- mysql_errno — Returns the numerical value of the error message from previous MySQL operation
- mysql_error — Returns the text of the error message from previous MySQL operation
- mysql_escape_string — Escapes a string for use in a mysql_query
- mysql_fetch_array — Fetch a result row as an associative array, a numeric array, or both
- mysql_fetch_assoc — Fetch a result row as an associative array
- mysql_fetch_field — Get column information from a result and return as an object
- mysql_fetch_lengths — Get the length of each output in a result
- mysql_fetch_object — Fetch a result row as an object
- mysql_fetch_row — Get a result row as an enumerated array
- mysql_field_flags — Get the flags associated with the specified field in a result
- mysql_field_len — Returns the length of the specified field
- mysql_field_name — Get the name of the specified field in a result
- mysql_field_seek — Set result pointer to a specified field offset
- mysql_field_table — Get name of the table the specified field is in
- mysql_field_type — Get the type of the specified field in a result
- mysql_free_result — Free result memory
- mysql_get_client_info — Get MySQL client info
- mysql_get_host_info — Get MySQL host info
- mysql_get_proto_info — Get MySQL protocol info
- mysql_get_server_info — Get MySQL server info
- mysql_info — Get information about the most recent query
- mysql_insert_id — Get the ID generated in the last query
- mysql_list_dbs — List databases available on a MySQL server
- mysql_list_fields — List MySQL table fields
- mysql_list_processes — List MySQL processes
- mysql_list_tables — List tables in a MySQL database
- mysql_num_fields — Get number of fields in result
- mysql_num_rows — Get number of rows in result
- mysql_pconnect — Open a persistent connection to a MySQL server
- mysql_ping — Ping a server connection or reconnect if there is no connection
- mysql_query — Send a MySQL query
- mysql_real_escape_string — Escapes special characters in a string for use in an SQL statement
- mysql_result — Get result data
- mysql_select_db — Select a MySQL database
- mysql_set_charset — Sets the client character set
- mysql_stat — Get current system status
- mysql_tablename — Get table name of field
- mysql_thread_id — Return the current thread ID
- mysql_unbuffered_query — Send an SQL query to MySQL without fetching and buffering the result rows

## php alone

```php
<?php
// 注释

/*
 * 多行注释
 */

$variable = 5;  # 弱类型语言

echo "xxx: $variable";  # 打印变量直接写
echo PHP_EOL;   # 换行符
echo "str" . $variable . "str2";
echo "a",1,2,3;
print "a123" # almost the same, but only take one param

var_dump($variable);    # return data type and value

# Boolean
true;
false;

# Array
$arr = array(1,2,3);
$arrlength = count($arr);

$arr = array("1"=>1);   # associative array
$arr['1'];

foreach($arr as $x => $x_val){
    //
}

array(
    array(1,2),
    array(2,3)
);

sort($arr); # ascend
rsort($arr);    # descend
asort($arr);    # ascend, val
ksort($arr);    # ascend, key
arsort($arr);   # descend, val
krsort($arr);   # descend, key

# Null
null;

# string func
strlen("a");    # 1
str_word_count("a b");    # 2, 空格分隔，词数
strrev("ab");   # ba, reverse
strpos("a b", "a");     # 0
strpos("a b", "c");     # false
str_replace("b", "c", "a b");   # a c

is_numeric("123");

# number func
PHP_INT_MAX, PHP_INT_MIN, PHP_INT_SIZE;
is_int($variable);  # below are aliases
is_integer($variable);
is_long($variable);
(int)$variable; # cast

PHP_FLOAT_MAX, PHP_FLOAT_MIN, PHP_FLOAT_DIG, PHP_FLOAT_EPSILON;
is_float($variable);
is_double($variable);   # alias
is_finite($variable);
is_infinite($variable);
is_nan($variable);
pi()

min(list);
max(list);
abs(-1);
sqrt(4);
$a ** $b;
round(0.4);
rand();
rand(10, 100);
$a === $b;  # check value and type
$a <> $b;   # !=
$a++;
$a += $b;
$a and $b;
$a or $b;
$a xor $b;
$x = expr1 ?? expr2;    # 根据expr1是否为null判断

if(condition){
    //
}
elseif(condition){
    //
}
else{
    //
}

switch($n) {
    case label1:
        //
        break;
    default:
        break;
}

while(condition){
    //
}

do{
    //
}while(condition);

for(;;){
    //
}

foreach($array as $value){
    //
}

# constant
define(name, value, bool case-insensitive);

function test($y){
    $GLOBALS['variable'] = 6;   # 使用全局变量
    static $x = 0;  # 不删除局部变量
    echo $y;
    return $x;
}

# Regex
preg_match($pattern, $str); # return 1 if pattern match else 0
preg_match_all($pattern, $str)  # return number of times matching
preg_replace($pattern, $str_replace, $str);

date(format[, timestamp]);
/*
 * format:
 * Y - year
 * m - month
 * d - day
 * l - day of the week
 * H - 24-hour format of hour
 * h - 12
 * i - minites
 * s - seconds
 * a - am or pm
 */

date_default_timezone_set("timezone");
mktime($hour, $minute, $second, $month, $day, $year);   # return the Unix timestamp for a date
strtotime($time[, $now]);   # convert a human readable string into Unix timestamp

include 'filename'; # warning if fail
require 'filename'; # fatal-error if fail

readfile("path");   # return file content
file_exists("path");    # return boolean,  whether file exists
$f = fopen("path", "mode");  # return file object
fread($f, filesize("path"));
fgets($f);  # read a single line
fgetc($f);  # read a single char
feof($f);   # check if EOF reached, return boolean
fclose($f);
/*
 * mode:
 * r
 * w
 * a, append
 * x, create new file for write only
 * + read/write
 */

// 上传文件
$_FILES["input name"]["name"];
$_FILES["input name"]["type"];
$_FILES["input name"]["size"];
$_FILES["input name"]["tmp_name"];
$_FILES["input name"]["error"];

/*
 * <form action="upload_file.php" method="post" enctype="multipart/form-data">
 *  <input type="filel" name="input name" id="file">
 *  <input type="submit" name="submit" value="提交">
 * </form>
 */

$from = $_FILES["input name"]["tmp_name"];
move_uploaded_file($from, $to);

// cookie
setcookie($name, $value, $expire, $path, $domain);   # 必须位于<html>之前
$expire = time() + 3600;    # 设置过期时间，单位s
$_COOKIE

// session
session_start();
// 存储 session 数据
$_SESSION['views']=1;
isset($_SESSION['views']);  # 是否存在
unset($_SESSION['views']);  # 销毁

session_destroy();
?>

