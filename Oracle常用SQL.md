## Oracle常用SQL

### Oracle中查询日期
```SQL
select * from gsppatchlog where to_number(to_char(deploytime,'YYYY'))=2019 and 
to_number(to_char(deploytime,'MM'))=2 and to_number(to_char(deploytime,'DD'))=27

select * from GSImageInterface where createtime > TO_DATE('2019/6/28  00:00:00', 'yyyy/MM/dd hh24:mi:ss')
```
### TEST为用户名，用户名必须是大写
```SQL
select * from all_tables where owner='TEST'；
```
### 查看当前登录的用户的表:
```SQL
select table_name from user_tables;
```
### 查询表中字段个数
```SQL
select count(distinct column_name)  from all_tab_cols where table_name = upper('imgioplogs')
```
### 查询表的详细信息
```SQL
select distinct table_name,column_name,data_type,DATA_length from all_tab_cols where table_name = upper('imgioplogs')
```

### 判断表是否存在，若存在则继续执行语句 
```SQL
DECLARE
    tableExist number;
BEGIN
    SELECT COUNT(1) INTO tableExist FROM user_tables WHERE table_name = upper('GSPAuditEvent');
    IF tableExist = 1 THEN
        DELETE FROM GSPAuditEvent WHERE ID = 'LSWLDWDQADD';
        INSERT INTO GSPAuditEvent (ID, NAME, Content, FunctionID, ISENABLE, CATEGORYID, ISPRECUT)
        VALUES ('LSWLDWDQADD', '往来单位地区新增', NULL, '往来单位地区定义', '1', 'LS', '0');
    END IF;
END;
```
### 判断表中某一字段是否存在，如果不存在则创建 
```SQL
DECLARE
    columnExist number;
BEGIN
    SELECT COUNT(1) INTO columnExist FROM USER_TAB_COLUMNS
    WHERE TABLE_NAME = UPPER('Banks') and column_name = UPPER('fullnameupper');
    IF columnExist = 0 THEN
    EXECUTE IMMEDIATE
        'alter table banks add (fullnameupper varchar2(100))';
    END IF;
END;
```
### 根据字段值查询表中字段
```
DECLARE  
CURSOR cur_query IS   
  SELECT COLUMN_NAME FROM USER_TAB_COLUMNS WHERE TABLE_NAME = 'LSWLDW';   
  a NUMBER;   
  sql_hard VARCHAR2(2000);   
  vv NUMBER;  
BEGIN   
  FOR rec1 IN cur_query LOOP  
  sql_hard := '';   
  sql_hard := 'SELECT count(*) FROM LSWLDW where '||rec1.COLUMN_NAME|| ' like ''63C0C4977C114096B925304C745EBB4D''';  
  EXECUTE IMMEDIATE sql_hard INTO vv;  
  IF vv > 0 THEN 
    insert into imgioplogs (logid,logtype) values('76EB8CED97104EF3AD8DCD373875808A',rec1.COLUMN_NAME);
  END IF;
  END LOOP;  
END;
select * from imgioplogs
```
### 根据字段值查询数据库中对应表的对应字段
```SQL
DECLARE  
CURSOR cur_query IS   
  SELECT table_name, column_name, data_type FROM user_tab_columns;   
  a NUMBER;   
  sql_hard VARCHAR2(2000);   
  vv NUMBER;  
BEGIN   
  FOR rec1 IN cur_query LOOP  
  a:=0;   
  IF rec1.data_type ='VARCHAR2' OR rec1.data_type='CHAR' THEN   
  a := 1;   
  END IF;   
  IF a>0 THEN   
  sql_hard := '';   
  sql_hard := 'SELECT count(*) FROM '|| rec1.table_name ||' where '   
  ||rec1.column_name|| ' like''吴芳''';--字段值   
  dbms_output.put_line(sql_hard);   
  EXECUTE IMMEDIATE sql_hard INTO vv;  
  IF vv > 0 THEN dbms_output.put_line('[字段值所在的表.字段]:['||rec1.table_name||'].['||rec1.column_name||']');   
  END IF;  
  END IF;  
  END LOOP;  
END;
```
### <span id="anchor">查询字段对应索引</span>
```SQL
SELECT b.uniqueness, a.index_name, a.table_name, a.column_name FROM all_ind_columns a, all_indexes b WHERE a.index_name=b.index_name AND a.table_name = upper('GSYIMAGEFILE') and a.column_name = upper('billid') ORDER BY a.table_name, a.index_name, a.column_position;

```

### 同时更新多个字段
```SQL
update A set (A.a2,A.a3) =(select B.b2,b.b3 from B where B.b1= A.a1 and A.a3=100) where A.a2 = '2'
```