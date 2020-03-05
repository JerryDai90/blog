# mysql 存储过程

好久没有写过存储过程了，记录一下关键语法规则。

## 游标的使用

**需要设置循环完成后执行的 SQL**

```sql
DECLARE done INT DEFAULT 0;
-- 只能定义一次，且在打开游标之前定义。（在游标定义之后才能定义）
DECLARE continue handler for not found set done = 1;
```
上面的意思是在出现 `not found` 这个错误的时候吧 `done` 变量设置成 1，需要加入这个的原因是因为遍历游标的时候，当变量到最后一行的时候这个时候会抛出错误，所以就需要扑捉此异常。（这里可以定义其他异常处理，包括捕捉到指定的异常码。）

**范例**

```sql
CREATE PROCEDURE `PRO_TEST`()
label_pro:begin

	DECLARE outside_val1 varchar(20);
	DECLARE outside_val2 varchar(20);
	   
	DECLARE inside_val1 varchar(20);
	DECLARE inside_val2 varchar(20);
        
	DECLARE done INT DEFAULT 0;

	--  外循环的游标
	declare cur_outside cursor for
		select va11, val2 from table1 ;
	--   内循环的游标
	declare cur_inside cursor for
		select va11, val2 from table2 ;  
    -- 定义错误处理        
	declare continue handler for not found SET done = 1;
	
	OPEN cur_outside; -- 开始读取游标
		read_loop_outside: LOOP
			FETCH cur_outside INTO outside_val1, outside_val2; -- 赋值
			-- 这里如果游标读取到最后一行后，就会设置 done 值为 1，这个时候就可以退出循环了。
			IF done THEN
				LEAVE read_loop_outside; -- 退出循环
			END IF;
			 
			-- -------------------------beging 嵌套游标循环 -------------------------
			open cur_inside;
						
				read_loop_inside: LOOP
					FETCH cur_inside INTO inside_val1, inside_val2;
					IF done THEN
						LEAVE read_loop_inside;
					END IF;
					-- do something	
						
				END LOOP; -- 结束内循环
			close cur_inside; -- 记得关闭，要不然第二次循环的时候会报错，说游标已经打开了		
			set done = 0; --  这里是关键，要不然 cur_inside 游标循环完成后就会出现退出情况了
			-- -------------------------end 嵌套游标循环 ----------------------------
			
			-- do something
			
		END LOOP; -- 结束外循环
	close cur_outside;
END
```
## 调试
可以直接使用 ` SELECT XXX` XXX 是变量名称（调试利器），执行的时候就会打印到控制台的了。

## 参考文献
https://segmentfault.com/a/1190000005807737

