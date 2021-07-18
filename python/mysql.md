
## explain
+----+-------------+----------+------+---------------+------+---------+------+------+-------+  
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra |  
+----+-------------+----------+------+---------------+------+---------+------+------+-------+  
|  1 | SIMPLE      | sys_user | ALL  | NULL          | NULL | NULL    | NULL |  903 | NULL  |  
+----+-------------+----------+------+---------------+------+---------+------+------+-------+  

+ id
+ select_type
+ table
+ type
  + 从最好到最差
  + system > const > eq_ref > ref > range > index > all 
  + system 表只有一行记录（等于系统表），是 const 的特例，平时不会出现可忽略不计
  + const 表示通过索引一次就找到了，const 用于通过 primaryKey / unique 索引，以为只匹配一行数据，
    所以很快，将主键置于 where 条件，mysql 就能将该查询转换为一个常量
  + eq_ref 唯一性索引扫描，对于每个索引键表中只有一条数据与之匹配，常见于主键或唯一性索引扫描 
  + ref 非唯一性索引扫描，即可能返回多条符合条件的数据
  + range 
  + index
  + all 
  + 一般来说得保证查询至少达到range级别，最好达到ref
+ possible_keys
+ key
+ key_len
+ ref
+ rows
+ Extra