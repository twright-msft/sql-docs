---
title: "TRY...CATCH (SQL Server PDW)"
ms.custom: na
ms.date: 07/27/2016
ms.reviewer: na
ms.suite: na
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: ca350427-f7c3-41dc-8e4e-90bdfe744fdc
caps.latest.revision: 11
author: BarbKess
---
# TRY...CATCH (SQL Server PDW)
Implements error handling for Transact\-SQL that is similar to the exception handling in the Microsoft Visual C# and Microsoft Visual C++ languages. A group of SQL statements can be enclosed in a TRY block. If an error occurs in the TRY block, control is passed to another group of statements that is enclosed in a CATCH block.  
  
![Topic link icon](../../mpp/sqlpdw/media/Topic_Link.gif "Topic_Link")[Syntax Conventions &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/syntax-conventions-sql-server-pdw.md)  
  
## Syntax  
  
```  
BEGIN TRY  
     { sql_statement | statement_block }  
END TRY  
BEGIN CATCH  
     [ { sql_statement | statement_block } ]  
END CATCH  
[ ; ]  
```  
  
## Arguments  
*sql_statement*  
Is any Transact\-SQL statement.  
  
*statement_block*  
Any group of Transact\-SQL statements in a batch or enclosed in a BEGIN…END block.  
  
## Remarks  
A TRY…CATCH construct catches all execution errors that have a severity higher than 10.  
  
A TRY block must be immediately followed by an associated CATCH block. Including any other statements between the END TRY and BEGIN CATCH statements generates a syntax error.  
  
A TRY…CATCH construct cannot span multiple batches. A TRY…CATCH construct cannot span multiple blocks of Transact\-SQL statements. For example, a TRY…CATCH construct cannot span two BEGIN…END blocks of Transact\-SQL statements and cannot span an IF…ELSE construct.  
  
If there are no errors in the code that is enclosed in a TRY block, when the last statement in the TRY block has finished running, control passes to the statement immediately after the associated END CATCH statement. If there is an error in the code that is enclosed in a TRY block, control passes to the first statement in the associated CATCH block. If the END CATCH statement is the last statement in a stored procedure, control is passed back to the statement that called the stored procedure.  
  
When the code in the CATCH block finishes, control passes to the statement immediately after the END CATCH statement. Errors trapped by a CATCH block are not returned to the calling application. If any part of the error information must be returned to the application, the code in the CATCH block must do so by using mechanisms such as SELECT result sets or the RAISERROR and PRINT statements.  
  
TRY…CATCH constructs can be nested. Either a TRY block or a CATCH block can contain nested TRY…CATCH constructs. For example, a CATCH block can contain an embedded TRY…CATCH construct to handle errors encountered by the CATCH code.  
  
Errors encountered in a CATCH block are treated like errors generated anywhere else. If the CATCH block contains a nested TRY…CATCH construct, any error in the nested TRY block will pass control to the nested CATCH block. If there is no nested TRY…CATCH construct, the error is passed back to the caller.  
  
TRY…CATCH constructs catch unhandled errors from stored procedures executed by the code in the TRY block. Alternatively, the stored procedures or triggers can contain their own TRY…CATCH constructs to handle errors generated by their code. For example, when a TRY block executes a stored procedure and an error occurs in the stored procedure, the error can be handled in the following ways:  
  
-   If the stored procedure does not contain its own TRY…CATCH construct, the error returns control to the CATCH block associated with the TRY block that contains the EXECUTE statement.  
  
-   If the stored procedure contains a TRY…CATCH construct, the error transfers control to the CATCH block in the stored procedure. When the CATCH block code finishes, control is passed back to the statement immediately after the EXECUTE statement that called the stored procedure.  
  
## Retrieving Error Information  
In the scope of a CATCH block, the following system functions can be used to obtain information about the error that caused the CATCH block to be executed:  
  
-   ERROR_NUMBER() returns the number of the error.  
  
-   ERROR_SEVERITY() returns the severity.  
  
-   ERROR_STATE() returns the error state number.  
  
-   ERROR_PROCEDURE() returns the name of the stored procedure or trigger where the error occurred.  
  
-   ERROR_MESSAGE() returns the complete text of the error message. The text includes the values supplied for any substitutable parameters, such as lengths, object names, or times.  
  
These functions return NULL if they are called outside the scope of the CATCH block. Error information can be retrieved by using these functions from anywhere within the scope of the CATCH block. For example, the following script shows a stored procedure that contains error-handling functions. In the `CATCH` block of a `TRY…CATCH` construct, the stored procedure is called and information about the error is returned.  
  
```  
-- Verify that the stored procedure does not already exist.  
IF OBJECT_ID ( 'usp_GetErrorInfo', 'P' ) IS NOT NULL   
    DROP PROCEDURE usp_GetErrorInfo;  
GO  
  
-- Create procedure to retrieve error information.  
CREATE PROCEDURE usp_GetErrorInfo  
AS  
SELECT  
    ERROR_NUMBER() AS ErrorNumber  
    ,ERROR_SEVERITY() AS ErrorSeverity  
    ,ERROR_STATE() AS ErrorState  
    ,ERROR_PROCEDURE() AS ErrorProcedure  
    ,ERROR_MESSAGE() AS ErrorMessage;  
GO  
  
BEGIN TRY  
    -- Generate divide-by-zero error.  
    SELECT 1/0;  
END TRY  
BEGIN CATCH  
    -- Execute error retrieval routine.  
    EXECUTE usp_GetErrorInfo;  
END CATCH;  
```  
  
## Errors Unaffected by a TRY…CATCH Construct  
TRY…CATCH constructs do not trap the following conditions:  
  
-   Warnings or informational messages that have a severity of 10 or lower.  
  
-   Attentions, such as client-interrupt requests or broken client connections.  
  
-   When the session is ended by a system administrator by using the KILL statement.  
  
The following types of errors are not handled by a CATCH block when they occur at the same level of execution as the TRY…CATCH construct:  
  
-   Compile errors, such as syntax errors, that prevent a batch from running.  
  
These errors are returned to the level that ran the batch, stored procedure, or trigger.  
  
Some errors that are compile errors in SQL Server (such as binding errors) are execution errors in PDW and they will be caught in a CATCH block.  
  
## Transaction Behavior  
The transactional behavior of **TRY**... **CATCH** blocks is not identical to that of the SQL Server. In SQL Server, when a **CATCH** block is entered, the transaction is uncommittable, but the data can be read. This state is characterized by the value of the **XACT_STATE** function of **-1**. While the state is **-1**, **SELECT** statements can be used to query data affected prior to an error being thrown.  
  
In PDW, there is a new state for the active transaction, in which data can be neither committed, nor accessed. This state can be identified by invoking the **XACT_STATE** function where -2 is the expected value for that state. We call this transaction state **rollback-only**.  
  
After entering a **CATCH** block and up until **ROLLBACK** is issued by the user or the end of batch is reached, no data-access statements are allowed in the batch (**SELECT**, DML, or DDL statements). The statements that can be executed within a transaction that reaches this state are **SET**, **DECLARE**, **PRINT**, **RAISERROR**, **THROW** and control-flow statements (**IF**/**ELSE**, **WHILE**, **TRY**…**CATCH**). This state can be identified by invoking the **XACT_STATE** function where -2 is the expected value for that state. The following example demonstrates the **XACT_STATE** behavior.  
  
|Step|Statement|XACT_STATE|Comments|  
|--------|-------------|---------------|------------|  
|1|BEGIN TRAN|0|Beginning transaction|  
|2|BEGIN TRY|1|Transaction is active.|  
|3|SELECT 1/0|1|Division by zero happens here.|  
||...|N/A|Control passed to CATCH block.|  
||END TRY|N/A||  
|4|BEGIN CATCH|-2|Transaction is now in a rollback-only state|  
|5|DECLARE @err int = @@ERROR|-2|DECLARE, SET, control flow statements are allowed.|  
|6|DECLARE @xact int = XACT_STATE()|-2|We cache error and transaction states for later use.|  
|6|END CATCH|-2||  
|7|-- SELECT @err, @xact|-2|SELECTs are still prohibited here.|  
|8|ROLLBACK|-2|Rolling back the transaction.|  
|9|SELECT @err, @xact|0|No active transactions, SELECTs are allowed.|  
  
Since the transactional behavior of **TRY**.. **CATCH** blocks has been altered in PDW, and that change is manifested and can be detected by the altered value of **XACT_STATE** function in the catch block (**-2** in PDW versus **-1** in SQL Server), it would be a good practice for scripts which handle errors to check for negative values of **XACT_STATE**, instead of specifically checking for **-1** or **-2**.  
  
For more information about uncommittable transactions and the **XACT_STATE** function, see [XACT_STATE &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/xact-state-sql-server-pdw.md).  
  
## Examples  
  
### A. Using TRY…CATCH  
The following example shows a `SELECT` statement that will generate a divide-by-zero error. The error causes execution to jump to the associated `CATCH` block.  
  
```  
BEGIN TRY  
    -- Generate a divide-by-zero error.  
    SELECT 1/0;  
END TRY  
BEGIN CATCH  
    SELECT  
        ERROR_NUMBER() AS ErrorNumber  
        ,ERROR_SEVERITY() AS ErrorSeverity  
        ,ERROR_STATE() AS ErrorState  
        ,ERROR_PROCEDURE() AS ErrorProcedure  
        ,ERROR_MESSAGE() AS ErrorMessage;  
END CATCH;  
GO  
```  
  
## See Also  
[XACT_STATE &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/xact-state-sql-server-pdw.md)  
[@@ERROR &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/error-sql-server-pdw.md)  
[ERROR_MESSAGE &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/error-message-sql-server-pdw.md)  
[ERROR_NUMBER &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/error-number-sql-server-pdw.md)  
[ERROR_PROCEDURE &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/error-procedure-sql-server-pdw.md)  
[ERROR_SEVERITY &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/error-severity-sql-server-pdw.md)  
[ERROR_STATE &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/error-state-sql-server-pdw.md)  
[RAISERROR &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/raiserror-sql-server-pdw.md)  
[THROW &#40;SQL Server PDW&#41;](../../mpp/sqlpdw/throw-sql-server-pdw.md)  
  