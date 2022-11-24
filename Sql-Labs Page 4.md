## Sql-Labs Page 4

### Less-54

这一关和前面形式上出现了区别，输入10次后会将内容重置

![image-20221121102214776](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121102214776.png)

这里的注入方法和第一关基本一样的，但是需要在最后的过程中找到==key==（类似于ctf）了。然后提交就可以了。

### Less-55

和前面的类似这一关的闭合是一个括号、

![image-20221121102500945](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121102500945.png)

### Less-56

和前两关类似，这一关主要是==单引号加括号的形式==

![image-20221121102644009](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121102644009.png)

### Less-57

这一关依然和前面的差不多，这次是==双引号闭合==

![image-20221121102833355](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121102833355.png)

### Less-58

这一关，我们查闭合容易查到。但是发现没有办法使用联合查询。查看源代码发现数据不是数据库中的，是数组中的。这里有报错信息显示，我们可以直接用报错注入。。

```sql
?id=1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='challenges'),0x7e),1)--+
```

后面的操作和前面的大体相同。

### Less-59

和上一关类似，这里是整数型

### Less-60

==双引号加括号==闭合

### Less-61

==单引号加两个括号==闭合

![image-20221121103919255](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121103919255.png)

### Less-62

没有报错信息显示，使用==布尔盲注==或者==时间盲注==

![image-20221121104327261](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121104327261.png)

闭合点是 ==单引号加一个括号==

### Less-63

和上一关类似，闭合点是==单引号==

![image-20221121104539978](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121104539978.png)

### Less-64

和前面的类似，闭合方式是==两个括号==

### Less-65

闭合方式是==双引号加一个括号==但是网上有博客说是==一个括号==。有些事直接看的sql语句那里，其实不光是那里，其他地方也对==id==进行了处理的。大家可以自己去找一下。

![image-20221121105626542](C:\Users\HP\AppData\Roaming\Typora\typora-user-images\image-20221121105626542.png)