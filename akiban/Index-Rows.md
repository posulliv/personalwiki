# Index Rows

salaries table in employees dataset
employee is parent of salaries table

index on salary column in that table

keys for that index look like:

 salary, emp_no, from_date

in general, index consists of
 - columns declared in the index
 - hKey columns that were declared as part of index

so now an execution plan that looks like:

```console
employees=> explain select to_date from salaries where salary > 38623;
                                                                       OPERATORS                                                                        
--------------------------------------------------------------------------------------------------------------------------------------------------------
 project([Field(3)])
   AncestorLookup_Default(Index(employees.salaries.sal_idx[IndexColumn(salary)]) -> [employees.salaries])
     IndexScan_Default(Index(employees.salaries.sal_idx[IndexColumn(salary)]) (>=UnboundExpressions[Literal(38623)],<=UnboundExpressions[Literal(38623)]))
(3 rows)

Time: 3.445 ms
employees=> 
```

means an index row will be found by the index lookup with a key that looks like:

```console
(long)38623,(long)253406,(long)1025108
```

the last 2 segments of this key is a hKey that can be used as input to the 
ancestor lookup operator.

Now lets say we want to join employees and salaries. Execution plan might look
like:

```console
employees=> explain select salaries.to_date, employees.last_name from employees join salaries on employees.emp_no = salaries.emp_no where salary = 38623;
                                                                          OPERATORS                                                                          
-------------------------------------------------------------------------------------------------------------------------------------------------------------
 project([Field(9), Field(3)])
   Flatten_HKeyOrdered(employees.employees INNER employees.salaries)
     AncestorLookup_Default(Index(employees.salaries.sal_idx[IndexColumn(salary)]) -> [employees.employees, employees.salaries])
       IndexScan_Default(Index(employees.salaries.sal_idx[IndexColumn(salary)]) (>=UnboundExpressions[Literal(38623)],<=UnboundExpressions[Literal(38623)]))
(4 rows)

Time: 7.879 ms
employees=>
```

In the above case, the index keys coming from the index scan are the same. 
Notice the ancestor lookup is now different and has multiple ancestor types:
1) emplyees and 2) salaries. So in this execution plan, 2 rows from the group
tree are being fetched i.e. 2 random probes into the group tree. The flatten
operator is then needed to combine these 2 rows.

Next, lets say we want to join with another branch in the group - titles.
Execution plan might look like:

```console
employees=> explain select salaries.to_date, employees.last_name, titles.title from employees join salaries on employees
                                                                           OPERATORS                                                                           
---------------------------------------------------------------------------------------------------------------------------------------------------------------
 project([Field(9), Field(3), Field(11)])
   Product_NestedLoops(flatten(employees.employees, employees.salaries) x flatten(employees.employees, employees.titles))
     Flatten_HKeyOrdered(employees.employees INNER employees.salaries)
       AncestorLookup_Default(Index(employees.salaries.sal_idx[IndexColumn(salary)]) -> [employees.employees, employees.salaries])
         IndexScan_Default(Index(employees.salaries.sal_idx[IndexColumn(salary)]) (>=UnboundExpressions[Literal(38623)],<=UnboundExpressions[Literal(38623)]))
     Filter_Default([flatten(employees.employees, employees.titles)])
       Flatten_HKeyOrdered(employees.employees INNER employees.titles)
         BranchLookup_Nested(_akiban_employees employees.employees -> employees.titles)
(8 rows)

Time: 3.498 ms
employees=> 
```

Again, the keys from the index scan are the same. As before, an ancestor
lookup is performed and then flattened because there are more than two
ancestor types. In this case however, we need to combine the rows from
two different branches in the group. This is what the product nested loops
operator is doing. For each row that comes out of the flatten operator in the 
above execution plan, a branch lookup is performed. This branch lookup
does a random probe into the tree for a certain emp_no. Then for that emp_no,
it performs a sequential scan of all titles associated with that emp_no. The
output will be a row with fields from the employees, titles, and salaries
tables.