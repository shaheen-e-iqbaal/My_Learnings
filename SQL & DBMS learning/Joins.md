
1. inner join (only include that rows where foreign key matches in both tables)
2. natural join
3. left outer join
4. right outer join
5. full outer join (simply union of left outer + right outer)
6. cross join (cartesian join)

Syntax of cross join : select * from table1 cross join tabel2;
Main point is that it doesn't require any "ON". but we can use "ON" also.

Similarly Natural join also doesn't require any "ON" condition. it simply does join based on the common column (name + datatype should be same) between two tables. so it used this common column as foreign key and then does inner join.