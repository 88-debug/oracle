学号：201810601109   姓名：马巧   班级：2018级软工2班

# 实验5：PL/SQL编程

## 实验目的

+ 了解PL/SQL语言结构
+ 了解PL/SQL变量和常量的声明和使用方法
+ 学习条件语句的使用方法
+ 学习分支语句的使用方法
+ 学习循环语句的使用方法
+ 学习常用的PL/SQL函数
+ 学习包，过程，函数的用法。

## - 实验场景：

+ 假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
+ 本实验以实验四为基础

## 实验内容：

1.创建一个包(Package)，包名是maqiao_Pack。

#### 代码如下：

```
   create or replace PACKAGE maqiao_Pack IS
  /*
  本实验以实验4为基础。
  包maqiao_Pack中有：
  一个函数:Get_SaleAmount(V_DEPARTMENT_ID NUMBER)，
  一个过程:Get_Employees(V_EMPLOYEE_ID NUMBER)
  */
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
  PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
END maqiao_Pack;
```

![](./01创建.png)

2.在maqiao_Pack中创建一个函数SaleAmount ，查询部门表，统计每个部门的销售总金额，每个部门的销售额是由该部门的员工(ORDERS.EMPLOYEE_ID)完成的销售额之和。函数SaleAmount要求输入的参数是部门号，输出部门的销售金额。在maqiao_Pack中创建一个过程，在过程中使用游标，递归查询某个员工及其所有下属，子下属员工。过程的输入参数是员工号，输出员工的ID,姓名，销售总金额。信息用dbms_output包中的put或者put_line函数。输出的员工信息用左添加空格的多少表示员工的层次（LEVEL）。比如下面显示5个员工的信息：

#### 代码如下

```
   create or replace PACKAGE BODY maqiao_Pack IS
  FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER
  AS
    N NUMBER(20,2); --注意，订单ORDERS.TRADE_RECEIVABLE的类型是NUMBER(8,2),汇总之后，数据要大得多。
    BEGIN
      SELECT SUM(O.TRADE_RECEIVABLE) into N  FROM ORDERS O,EMPLOYEES E
      WHERE O.EMPLOYEE_ID=E.EMPLOYEE_ID AND E.DEPARTMENT_ID =V_DEPARTMENT_ID;
      RETURN N;
    END;

  PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
  AS
    LEFTSPACE VARCHAR(2000);
    begin
      --通过LEVEL判断递归的级别
      LEFTSPACE:=' ';
      --使用游标
      for v in
      (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
      START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
      CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
      LOOP
        DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                             V.EMPLOYEE_ID||' '||v.NAME);
      END LOOP;
    END;
END maqiao_Pack;
```



![](./02建包.png)

# 测试

## + 函数Get_SaleAmount()测试方法：

```
select count(*) from orders;
select maqiao_Pack.Get_SaleAmount(11) AS 部门11应收金额,maqiao_Pack.Get_SaleAmount(12) AS 部门12应收金额 from dual;
```

![](./03查询.png)

![](./04.png)

### 过程Get_Employees()测试代码：

```
   set serveroutput on
DECLARE
  V_EMPLOYEE_ID NUMBER;    
BEGIN
  V_EMPLOYEE_ID := 1;
  maqiao_Pack.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;  
  V_EMPLOYEE_ID := 11;
  maqiao_Pack.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;    
END;
```



![](./05.png)

#### 由于订单只是按日期分区的，上述统计是全表搜索，因此统计速度会比较慢，如何提高统计的速度呢？

+ 用Where子句替换HAVING子句
+ 避免使用HAVING子句, HAVING 只会在检索出所有记录之后才对结果集进行过滤。
+ 这个处理需要排序,总计等操作。 如果能通过WHERE子句限制记录的数目,那就能减少这方面的开销. (非oracle中)on、where、having这三个都可以加条件的子句中，on是最先执行，where次之，having最后，因为on是先把不符合条件的记录过滤后才进行统计，它就可以减少中间运算要处理的数据，按理说应该速度是最快的，where也应该比having快点的，因为它过滤数据后才进行sum，在两个表联接时才用on的.
+ 所以在一个表的时候，就剩下where跟having比较了。在这单表查询统计的情况下，如果要过滤的条件没有涉及到要计算字段，那它们的结果是一样的，只是where可以使用rushmore技术，而having就不能，在速度上后者要慢如果要涉及到计算的字段，就表示在没计算之前，这个字段的值是不确定的，根据上篇写的工作流程，where的作用时间是在计算之前就完成的，而having就是在计算后才起作用的，所以在这种情况下，两者的结果会不同。
+ 在多表联接查询时，on比where更早起作用。系统首先根据各个表之间的联接条件，把多个表合成一个临时表后，再由where进行过滤，然后再计算，计算完后再由having进行过滤。

### 实验总结

PL/SQL程序由三个块组成，即声明部分、执行部分、异常处理部分。  

其结构如下：

```
1   DECLARE   
2   --声明部分: 在此声明PL/SQL用到的变量,类型及游标，以及局部的存储过程和函数 
3   BEGIN 
4     -- 执行部分:  过程及SQL 语句  , 即程序的主要部分 
5   EXCEPTION 
6     -- 执行异常部分: 错误处理 
7   END;
```

PL/SQL块可以分为以下几类：

1. 无名块或匿名块（anonymous）：动态构造，只能执行一次，可调用其它程序，但不能被其它程序调用。

2. 命名块（named）：是带有名称的匿名块，这个名称就是标签。

3. 子程序（subprogram）：存储在数据库中的存储过程、函数等。当在数据库上建立好后可以在其它程序中调用它们。

4. 触发器（Trigger）：当数据库发生操作时，会触发一些事件，从而自动执行相应的程序。

5. 程序包/包（package）：存储在数据库中的一组子程序、变量定义。在包中的子程序可以被其它程序包或子程序调用。但如果声明的是局部子程序，则只能在定义该局部子程序的块中调用该局部子程序。

PL/SQL结构

1.PL/SQL块中可以包含子块。  
2.子块可以位于 PL/SQL中的任何部分。  
3.子块也即PL/SQL中的一条命令。

