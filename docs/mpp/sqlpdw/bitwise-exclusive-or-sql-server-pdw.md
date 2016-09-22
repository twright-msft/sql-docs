---
title: "^ (Bitwise Exclusive OR) (SQL Server PDW)"
ms.custom: na
ms.date: 07/27/2016
ms.reviewer: na
ms.suite: na
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: c2b3f99c-0590-46be-a4cc-08d23fece005
caps.latest.revision: 8
author: BarbKess
---
# ^ (Bitwise Exclusive OR) (SQL Server PDW)
Performs a bitwise exclusive OR operation between two integer values.  
  
![Topic link icon](../../mpp/sqlpdw/media/Topic_Link.gif "Topic_Link")[Syntax Conventions &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/syntax-conventions-sql-server-pdw.md)  
  
## Syntax  
  
```  
expression ^expression  
```  
  
## Arguments  
*expression*  
Is any valid [expression](../../mpp/sqlpdw/expressions-sql-server-pdw.md) of any one of the data types of the integer data type category, or the **bit**, or the **binary** or **varbinary** data types. *expression* is treated as a binary number for the bitwise operation.  
  
> [!NOTE]  
> Only one *expression* can be of either **binary** or **varbinary** data type in a bitwise operation.  
  
## Result Types  
**int** if the input values are **int**.  
  
**smallint** if the input values are **smallint**.  
  
**tinyint** if the input values are **tinyint**.  
  
## Remarks  
The **^** bitwise operator performs a bitwise logical exclusive OR between the two expressions, taking each corresponding bit for both expressions. The bits in the result are set to 1 if either (but not both) bits (for the current bit being resolved) in the input expressions have a value of 1. If both bits are 0 or both bits are 1, the bit in the result is cleared to a value of 0.  
  
If the left and right expressions have different integer data types (for example, the left *expression* is **smallint** and the right *expression* is **int**), the argument of the smaller data type is converted to the larger data type. In this case, the **smallint***expression* is converted to an **int**.  
  
## Examples  
The following example creates a table using the **int** data type to store the original values and inserts two values into one row.  
  
```  
CREATE TABLE bitwise  
(   
a_int_value int NOT NULL,  
b_int_value int NOT NULL  
);  
GO  
INSERT bitwise VALUES (170, 75);  
GO  
```  
  
The following query performs the bitwise exclusive OR on the `a_int_value` and `b_int_value` columns.  
  
```  
SELECT a_int_value ^ b_int_value  
FROM bitwise;  
GO  
```  
  
Here is the result set:  
  
```  
-----------   
225           
  
(1 row(s) affected)  
```  
  
The binary representation of 170 (`a_int_value` or `A`) is `0000 0000 1010 1010`. The binary representation of 75 (`b_int_value` or `B`) is `0000 0000 0100 1011`. Performing the bitwise exclusive OR operation on these two values produces the binary result `0000 0000 1110 0001`, which is decimal 225.  
  
```  
(A ^ B)     
         0000 0000 1010 1010  
         0000 0000 0100 1011  
         -------------------  
         0000 0000 1110 0001  
```  
  
## See Also  
[Common Metadata Query Examples &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/common-metadata-query-examples-sql-server-pdw.md)  
[Expressions &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/expressions-sql-server-pdw.md)  
[Operators &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/operators-sql-server-pdw.md)  
[Bitwise Operators &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/bitwise-operators-sql-server-pdw.md)  
[^= &#40;Bitwise Exclusive OR EQUALS&#41; &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/bitwise-exclusive-or-equals-sql-server-pdw.md)  
  