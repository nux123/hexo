title: oracle取第一条数据
date: 2015-04-17 09:44:40
tags:
---

问题：在项目中有一张设备检测信息表DEVICE_INFO_TBL, 每个设备每天都会产生一条检测信息，现在需要从该表中检索出每个设备的最新检测信息。也就是device_id字段不能重复，消除device_id字段重复的记录，而且device_id对应的检测信息test_result是最新的。
(本日志为转载, 找不到源地址, 谢原作者)
 

解决思路：用Oracle的row_number() over函数来解决该问题。

解决过程：

 1.查看表中的重复记录
```{bash}
select
    t.id,
    t.device_id,
    t.update_dtm,
    t.test_result
from DEVICE_INFO_TBL t
```
![01](/img/01.jpg)

2.标记重复的记录
```{bash}
select
    t.id,
    t.device_id,
    t.update_dtm,
    t.test_result,
    row_number() OVER(PARTITION BY device_id ORDER BY t.update_dtm desc) as row_flg   
from DEVICE_INFO_TBL t
```
![02](/img/02.jpg)


```{bash}
select
    temp.id,
    temp.device_id,
    temp.update_dtm,
    temp.test_result
from (
         select
             t.id,
             t.device_id,
             t.update_dtm,
             t.test_result,
             row_number() OVER(PARTITION BY device_id ORDER BY t.update_dtm desc) as row_flg   
          from DEVICE_INFO_TBL t ) temp
where temp.row_flg  = '1'
```
![03](/img/03.jpg)
row_number() OVER (PARTITION BY COL1 ORDER BY COL2) 表示根据COL1分组，在分组内部根据 COL2排序，而此函数计算的值就表示每组内部排序后的顺序编号（组内连续的唯一的).

　　与rownum的区别在于：使用rownum进行排序的时候是先对结果集加入伪列rownum然后再进行排序，而此函数在包含排序从句后是先排序再计算行号码．

　　row_number()和rownum差不多，功能更强一点（可以在各个分组内从1开时排序）．

　　rank()是跳跃排序，有两个第二名时接下来就是第四名（同样是在各个分组内）．

　　dense_rank()l是连续排序，有两个第二名时仍然跟着第三名。相比之下row_number是没有重复值的 ．

　　lag（arg1,arg2,arg3):
arg1是从其他行返回的表达式
arg2是希望检索的当前行分区的偏移量。是一个正的偏移量，时一个往回检索以前的行的数目。
arg3是在arg2表示的数目超出了分组的范围时返回的值。