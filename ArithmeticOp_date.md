# Arithmetic Operations concepts on Date in SQL
## How to define the variable in SQL?
  * To do that we need to use the ```DECLARE``` keyword.
  * Syntax:
    ~~~~sql
    DECLARE @date1 datetime = '2023-01-01';
    DECLARE @date2 datetime = '2022-01-01';
    ~~~~
 * Adding and subtracting two dates may give unexpected results.
 * In SQL server, the date first converted to a corrosponding integer values and then the operation performed. For example;
   ~~~sql
