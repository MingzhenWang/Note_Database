## SQLserver常用SQL

### 首先判断是否存在表，若存在则创建
```SQL
IF EXISTS (SELECT 1 FROM sysobjects WHERE name = 'GSPAuditEvent' AND type = 'U')
BEGIN
    DELETE FROM GSPAuditEvent WHERE ID = 'LSWLDWDQADD';
    INSERT INTO GSPAuditEvent (ID, NAME, Content, FunctionID, ISENABLE, CATEGORYID, ISPRECUT)
        VALUES ('LSWLDWDQADD', '往来单位地区新增', NULL, '往来单位地区定义', '1', 'LS', '0');
END
```

### 首先判断表中是否存在某字段，如果不存在则创建 
```SQL
IF NOT exists
(SELECT * FROM SYSCOLUMNS WHERE ID = OBJECT_ID('Banks') AND NAME = 'FullNameUpper')
ALTER TABLE Banks ADD FullNameUpper VARCHAR(60)
```
### 查询数据库中表的个数
```SQL
select count(*) from sysobjects where type='U'
```
### 查询表中字段的个数
```SQL
select count(*) from syscolumns where id = object_id('表名')
```

### 修改表名
```SQL
execute sp_rename '旧表名','新表名'
```
### 游标
```SQL
BEGIN
DECLARE @DWBH varchar(30)
DECLARE @YHZH varchar(128)
DECLARE @YHZHCount int 
DECLARE My_Cursor CURSOR FOR (select LSWLDW_WLDWBH from LSWLDW )
OPEN My_Cursor;
FETCH NEXT FROM My_Cursor INTO @DWBH;
WHILE @@FETCH_STATUS = 0
    BEGIN
        PRINT @DWBH;
				select @YHZHCount = count(*) from BankAccounts where OFOBJECT = @DWBH;
				if (@YHZHCount > 1)
				BEGIN
				update BankAccounts set ismain = '0' where OFOBJECT = @DWBH;
				select @YHZH = BankAccountID from BankAccounts where BankAccountID = (select top 1 BankAccountID from BankAccounts where OFOBJECT = @DWBH);
				update BankAccounts set ismain = '1' where BankAccountID = @YHZH;
				update lswldw set LSWLDW_YHZH = (select BankAccountCode from BankAccounts where BankAccountID = @YHZH) where LSWLDW_WLDWBH = @DWBH;
				END;
        FETCH NEXT FROM My_Cursor INTO @DWBH
    END
CLOSE My_Cursor; 
DEALLOCATE My_Cursor; 
end
```


### SQL Server 根据另一张表更新表
```SQL
update table1 set field1=table2.field1,field2=table2.field2 from table2 where table1.id=table2.id
```



### <span id="anchor">查询字段对应索引</span>
```SQL
SELECT a.name as [索引名称]
,c.name as [表名]
,d.name as [索引字段名]
,d.colid as [索引字段位置]
FROM sysindexes a(nolock)
JOIN sysindexkeys b(nolock) ON a.id=b.id AND a.indid=b.indid
JOIN sysobjects c(nolock) ON b.id=c.id
JOIN syscolumns d(nolock) ON b.id=d.id AND b.colid=d.colid
WHERE a.indid NOT IN(0,255)
AND c.name='GSYIMAGEFILE' and d.name ='BILLNM'
```