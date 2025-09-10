
-> There are two types of relational Databases. 1. OLTP (Online Transaction processing) 2. OLAP (Online Analytics processing)


-> When we have simple write heavy type queries then OLTP is good choice. but when we have complex read queries, then OLAP is good choice.

-> Tuple (Row) oriented (also called NSM) database falls under OLTP.

-> why OLTP is not good for simple write? OLTP stores data in tuple oriented storage ex: postgres. so it is good for read queries where we need entire tuple. but when need multiple single attribute from multiple tuple then we have to do multiple I/O to collect this data. and due to the tuple storage, we will get unnecessary attributes also because we can read entire page not just required attribute. 

Advantages of OLTP : Fast insert, update, delete. good for queries that need entire tuple 
Disadvantages of OLTP : Bad when we need to scan large portion of table or need handful of attributes. Inefficient storage in pages as each tuple can have different size. difficult to apply compression.


OLAP : 

-> Column oriented (also called DSM) database falls under OLAP.

-> In such databases, we store data as column wise. i.e. in each page, it will contain the value of single attribute of maximum rows. if single page can't store, the new page will be inserted. but main point is that, only single column value will be there in page.

-> Advantages : complex reads based on column values are fast. compression can be really good.
-> Disadvantages : insert/delete/update query on row can be really slow as it will need to require I/O for all pages which stores different columns values. 