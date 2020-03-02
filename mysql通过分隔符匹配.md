# mysql通过分隔符匹配

存储的时候，一对多的时候，一个字段使用逗号分隔多个值，发现使用 INSTR 会出现问题，比如
admin1,admin2 这个时候如果使用 INSTR 来匹配是会匹配上的。这个时候就需要自定义函数来切割数据。因此就有了如下的函数（老夫已经多年没有写过 mysql 的函数了。如有 bug，那你来打我呀。）

```sql
-- 增加了函数判断方法
DROP function IF EXISTS `F_INSTR_FOR_SEPARATOR`;

DELIMITER $$
CREATE FUNCTION `F_INSTR_FOR_SEPARATOR`(
    userIds varchar(256),
    matchUserId varchar(256),
    _separator varchar(10)
) RETURNS varchar(256) CHARSET utf8
BEGIN
    -- 待匹配的字符
    declare tempUserIds varchar(256) default userIds;
    -- 循环次数
    declare _count INTEGER;
    -- 分隔出来的字符
    declare _tempUserId varchar(256);
    -- 是否匹配了
    
    declare _isMatch INTEGER default 0;
    -- 定义分隔符好
    declare _tempSeparator varchar(10) default ',';
    
    -- 获取里面有多少个分隔符，用于循环
    select 1+(length(userIds) - length(replace(userIds,_tempSeparator,''))) into _count;
    
    WHILE _count >= 0 DO
    
        -- 找到第一个被分隔的字符进行匹配
        SELECT SUBSTRING_INDEX(tempUserIds, _tempSeparator, 1) into _tempUserId;
        
        if matchUserId = _tempUserId then 
			set _isMatch = 1;
            
        END IF;
        -- 去掉上面已经批评过的字符
        SELECT SUBSTRING(tempUserIds, INSTR(tempUserIds,_tempSeparator)+1) into tempUserIds;
        
        set _count = _count-1;
        
    END WHILE;
    
RETURN _isMatch;
END$$

DELIMITER ;
```



