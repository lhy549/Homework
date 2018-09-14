# 总结分割方法:
**思路：**
去重复后的lagou_position表内的公司、城市和地区、及其他字段分离成三个表(lagou_company,lagou_city,lagou_position),城市表和china_city链接，


1.**获取公司有关的字段company_id,company_short_name,company_full_name,company_size,financestage去重复， 把这些数据插入到一个新的表格lagou_company中**
```
create table lagou_company as 
  select distinct company_id cid,company_short_name short_name,company_full_name full_name,
      company_size size,financestage from lagou;
```

2.**删除lagou表原来关于公司内容的字段company_short_name、company_full_name、company_size、financestage，只保留company_id**
**示例：**
```
--删除原表字段
alter table lagou drop company_short_name;
alter table lagou drop company_full_name;
alter table lagou drop company_size;
alter table lagou drop financestage;
```

3.**根据p_province表获得区id、省、市、区的数据表**
获取china_city全国表中的市的数量(通过depth这个字段就可以获取，depth代表级别之意)
-- select count(*) from china_city where depth = 2;  -- 373 市  
-- select id,cityName from china_city where depth = 2; 


4.**根据p_province表获得市id、省、市、null的数据表**
-- 同样的，获取china_city全国表中的县(区)的数量   -- 3341 县  (县和市总和 3714)
-- select count(*) from china_city where depth = 3; 
-- select id,cityName from china_city where depth = 3; 


5.**创建lagou_city表，把步骤3和4的数据插入进去，获得一个包含省市区和省市的表，数据有冗余但是方便查询，按需求来设计表。**

```
create table lagou_city as 
-- select x.id,d.cityName province,c.cityName city,x.cityName district from 
-- (select * from china_city where depth = 2) c
--	 join china_city x on c.id = x.parentId and x.depth = 3
--	 join china_city d on c.parentId = d.id and d.depth = 1;
```
6.**由于lagou表中的字段city（城市）和district(地区)的字段存在冗余，并且不能包含所有的城市和地区，所以这两个字段不能分割出来作为一个表来使用。**

7.**但是lagou表中的city和district这两个字段是不能保留的，假设我们要查询某个省的招聘信息，很明显这两个字段无法满足我们的要求，所以我们要把他们 更改成lagou_city表中其对应的id**


8.**把city和district更换成lagou_city的id时，我们要考虑一个问题，就是当district的字段为空时，我们获得id就是city（城市）的id，如果district(地区) 字段不为空时，我们获得的id就是district(地区)的id,这是两种截然不同的条件，所以我们可分开查询。**

9.**分两种情况lagou表中district为null时,使用lagou表和lagou_city表连接查询，需要获得的是city（城市）的id,但是条件限制一定要确定清楚。首先肯定是lagou和lagou_city中 的district（地区）的值一定是为空的。其次我们lagou_city表中的所有的城市都带有“市”，而lagou表中的城市则不带“市”，这也是一个要解决的问题（使用like关键词）。所以当前两个表连接查询的条件就是要解决这些问题。**

lagou表中的district不为null时，要获得是district的id。lagou表和lagou_city表的连接条件是二者的city一样，district(县)相同。 


