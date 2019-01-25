---
title: mysql分页列表使用LEFT JOIN时的优化方法
categories: 数据库
tags: mysql分页列表优化
copyright: true
---

# mysql分页列表使用LEFT JOIN时的优化方法

### 分页列表查询一般格式
* 取分页数据
```mysql
SELECT *
FROM table1 t1
LEFT JOIN table2 t2 ON t2.id = t1.id
LEFT JOIN table3 t3 ON t3.id = t1.id
LEFT JOIN table4 t4 ON t4.id = t1.id
WHERE t1.id = 1 AND t2. ...
LIMIT 0,30
```
* 取总数
```mysql
SELECT COUNT(0)
FROM table1 t1
LEFT JOIN table2 t2 ON t2.id = t1.id
LEFT JOIN table3 t3 ON t3.id = t1.id
LEFT JOIN table4 t4 ON t4.id = t1.id
WHERE t1.id = 1 AND t2. ...
```
#### 缺点
>* `LEFT JOIN`消耗性能
>* 全部`LEFT JOIN`后再取分页
>* 取总数时候不必要的`LEFT JOIN`影响性能

#### 优化
>* `WHERE` 条件语句用不到的`LEFT JOIN`的表，放到取分页后面
>* 取总数时候不必要的`LEFT JOIN`不要

* 取分页数据
```mysql
SELECT *
FROM (
    SELECT *
    FROM table1 t1
    LEFT JOIN table2 t2 ON t2.id = t1.id
    LEFT JOIN table3 t3 ON t3.id = t1.id
    WHERE t1.id = 1 AND t2. ...
    LIMIT 0,30
    )t
LEFT JOIN table4 t4 ON t4.id = t1.id
    
```
* 取总数
```mysql
SELECT COUNT(0)
FROM table1 t1
LEFT JOIN table2 t2 ON t2.id = t1.id
LEFT JOIN table2 t3 ON t3.id = t1.id
WHERE t1.id = 1 AND t2. ...
```
### 具体实际应用
#### 网站分页列表
##### 优化前
>* 数据量：9741 
>* 每页显示30条数据
>* 平均每页刷新耗时：1100ms
* 列表数据
```mysql
SELECT 
    w.*,
    d.`name` AS deptName,
    IFNULL(de.`name`,'') AS superviseName,
    CASE WHEN w.school_id IS NULL THEN d.`name` ELSE s.name END AS schoolName
    FROM website w
    LEFT JOIN sys_dept d ON d.id = w.dept_id
    LEFT JOIN sys_dept de ON de.id = w.supervise_id
    LEFT JOIN sys_dept scan ON scan.id = w.scan_id
    LEFT JOIN sys_dept verify ON verify.id = w.verify_id
    LEFT JOIN sys_dept_info i ON i.dept_id = w.dept_id
    LEFT JOIN dictionary_item item ON i.area_id = item.id
    LEFT JOIN school s ON s.id = w.school_id
    WHERE w.is_del = 0
    <if test="deptId != null and deptId != ''">
        AND d.id = #{deptId}
    </if>
    <if test="name != null and name != ''">
        AND (UPPER(w.name) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%')))
        OR UPPER(w.py) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%'))))
    </if>
    <if test="url != null and url != ''">
        AND UPPER(w.url) LIKE UPPER(CONCAT('%',CONCAT(#{url},'%')))
    </if>
    <if test="linkman != null and linkman != ''">
        AND UPPER(w.contact_name) LIKE UPPER(CONCAT('%',CONCAT(#{linkman},'%')))
    </if>
    ORDER BY w.create_date DESC
    <if test="pageSize != null ">
        LIMIT #{offset}, #{pageSize}
    </if>
```
* 网站数量
```mysql
SELECT 
    COUNT(0)
    FROM website w
    LEFT JOIN sys_dept d ON d.id = w.dept_id
    LEFT JOIN sys_dept de ON de.id = w.supervise_id
    LEFT JOIN sys_dept scan ON scan.id = w.scan_id
    LEFT JOIN sys_dept verify ON verify.id = w.verify_id
    LEFT JOIN sys_dept_info i ON i.dept_id = w.dept_id
    LEFT JOIN dictionary_item item ON i.area_id = item.id
    LEFT JOIN school s ON s.id = w.school_id
    WHERE w.is_del = 0
    <if test="deptId != null and deptId != ''">
        AND d.id = #{deptId}
    </if>
    <if test="name != null and name != ''">
        AND (UPPER(w.name) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%')))
        OR UPPER(w.py) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%'))))
    </if>
    <if test="url != null and url != ''">
        AND UPPER(w.url) LIKE UPPER(CONCAT('%',CONCAT(#{url},'%')))
    </if>
    <if test="linkman != null and linkman != ''">
        AND UPPER(w.contact_name) LIKE UPPER(CONCAT('%',CONCAT(#{linkman},'%')))
    </if>
```



##### 优化
>* 数据量：9741 
>* 每页显示300条数据
>* 平均每页刷新耗时：120ms

>* 每页显示30条数据
>* 平均每页刷新耗时：50ms
* 列表数据
```mysql
SELECT
    t.*,
    item.name                 AS areaName,
    IFNULL(scan.`name`, '')   AS scanName,
    IFNULL(verify.`name`, '') AS verifyName,
    de.`name`                 AS superviseName
    FROM (
        SELECT
            w.id,
            w.name,
            w.url,
            w.dept_id,
            w.scan_id,
            w.supervise_id,
            w.verify_id,
            CASE WHEN w.school_id IS NULL THEN d.`name` ELSE s.name END  AS schoolName,
            d.`name`                                                     AS deptName
        FROM website w
        LEFT JOIN sys_dept d ON d.id = w.dept_id
        LEFT JOIN school s ON s.id = w.school_id
        WHERE w.is_del = 0
        <if test="deptId != null and deptId != ''">
            AND d.id = #{deptId}
        </if>
        <if test="deptName != null and deptName != ''">
            AND (UPPER(d.name) LIKE UPPER(CONCAT('%',CONCAT(#{deptName},'%')))
            OR UPPER(d.py) LIKE UPPER(CONCAT('%',CONCAT(#{deptName},'%')))
            OR UPPER(s.name) LIKE UPPER(CONCAT('%',CONCAT(#{deptName},'%'))))
        </if>
        <if test="name != null and name != ''">
            AND (UPPER(w.name) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%')))
            OR UPPER(w.py) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%'))))
        </if>
        <if test="url != null and url != ''">
            AND UPPER(w.url) LIKE UPPER(CONCAT('%',CONCAT(#{url},'%')))
        </if>
        <if test="linkman != null and linkman != ''">
            AND UPPER(w.contact_name) LIKE UPPER(CONCAT('%',CONCAT(#{linkman},'%')))
        </if>
        ORDER BY w.create_date DESC
        <if test="pageSize != null ">
            LIMIT #{offset}, #{pageSize}
        </if>
    ) t
    LEFT JOIN sys_dept_info i ON i.dept_id = t.dept_id
    LEFT JOIN dictionary_item item ON i.area_id = item.id
    LEFT JOIN sys_dept de ON de.id = t.supervise_id
    LEFT JOIN sys_dept scan ON scan.id = t.scan_id
    LEFT JOIN sys_dept verify ON verify.id = t.verify_id
```
* 网站数量
```mysql
SELECT
    COUNT(0)
    FROM (
        SELECT
            w.id,
            w.name,
            w.url,
            w.dept_id,
            w.scan_id,
            w.supervise_id,
            w.verify_id,
            CASE WHEN w.school_id IS NULL THEN d.`name` ELSE s.name END  AS schoolName,
            d.`name`                                                     AS deptName
        FROM website w
        LEFT JOIN sys_dept d ON d.id = w.dept_id
        LEFT JOIN school s ON s.id = w.school_id
        WHERE w.is_del = 0
        <if test="deptId != null and deptId != ''">
            AND d.id = #{deptId}
        </if>
        <if test="deptName != null and deptName != ''">
            AND (UPPER(d.name) LIKE UPPER(CONCAT('%',CONCAT(#{deptName},'%')))
            OR UPPER(d.py) LIKE UPPER(CONCAT('%',CONCAT(#{deptName},'%')))
            OR UPPER(s.name) LIKE UPPER(CONCAT('%',CONCAT(#{deptName},'%'))))
        </if>
        <if test="name != null and name != ''">
            AND (UPPER(w.name) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%')))
            OR UPPER(w.py) LIKE UPPER(CONCAT('%',CONCAT(#{name},'%'))))
        </if>
        <if test="url != null and url != ''">
            AND UPPER(w.url) LIKE UPPER(CONCAT('%',CONCAT(#{url},'%')))
        </if>
        <if test="linkman != null and linkman != ''">
            AND UPPER(w.contact_name) LIKE UPPER(CONCAT('%',CONCAT(#{linkman},'%')))
        </if>
        ORDER BY w.create_date DESC
        <if test="pageSize != null ">
            LIMIT #{offset}, #{pageSize}
        </if>
    ) t
```
