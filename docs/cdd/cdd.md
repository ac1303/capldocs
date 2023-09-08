# 目录结构

> 该文档主要用于记录CDD文件xml数据所代表的意义



## DATATYPES（数据类型）

### IDENT 

> 对应CDD中的Raw Value类型

属性：

* id 该数据类型的ID
* oid 
* bm （猜测Bit Mask）

#### NAME

> 大概是保存不同语言下的名字？

##### TUV

> 未知

#### QUAL 

> 对应Qualifier

#### CVALUETYPE

> 该信号的数据信息？？？

属性：

* bl  数据长度（Bit Length）

* bo 

* enc 编码方式 （Encoding）

* sig  

* df 显示格式（Display Format）

* qty （猜测是选择Bit Length/Field Size中的某个选项）

* sz 

* minsz 

* maxsz 

##### UNIT
> 单位

#### PVALUETYPE

> 未知，但是目前结论是和CVALUETYPE一模一样



### LINCOMP

>  对应CDD中的Linear类型

> NAME、QUAL、CVALUETYPE、PVALUETYPE同IDENT类型

#### EXCL

> 保存无效值数据

属性：

* s   开始值（start）
* e   结束之（end）
* inv  无效性（invaliddity）

#### COMP

> 信号的计算方式

属性：

* s 开始值（对应 Lower Limit）
* e 结束值（对应Upper Limit）
* f  Factor 
* 0 Offset