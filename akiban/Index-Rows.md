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