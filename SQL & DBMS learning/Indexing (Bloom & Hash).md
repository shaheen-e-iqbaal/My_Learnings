
-> PostgreSQL supports multiple type of indexing. Bloom index and hash index are one of them. postgres uses B+ Tree as default index data structure.


Bloom Filter : It is a probabilistic data structures which checks availability of something. if it says, it is not available then it won't be present in database 100%. but if says, it is available, it might or might not be available in database and we will need to check database for it's availability confirmation.
-> Use [This article](https://www.percona.com/blog/bloom-indexes-in-postgresql/) to understand Bloom filter data structures. 
-> It is array of bits, so it can be stored in very tiny less space.


Bloom Index : using below query, we can build bloom index in postgres. 

Query : 
```
CREATE TABLE documents (
  id serial PRIMARY KEY,
  title text,
  author text,
  content text
);

-- Index multiple columns
CREATE INDEX doc_bloom_idx ON documents
USING bloom (title, author, content)
WITH (length=128, title=2, author=2, content=4);

-- Query
SELECT * FROM documents
WHERE author = 'Alice' AND title = 'Postgres';
```

-> Postgres will create bit array for each row of table. you can specify the length of this bit array for each column. in above query, length = 128 shows the size of bit array for bloom index for each row. this bit array is called signature of that row.

-> It will store in bloom index like Signature -> TID.

-> now, when you query like in above ex, it will first signature using author = "Alice" and title = 'Postgres', then it will scan sequentially. it will compare this query signature with all row signature, and will collect all TID for which it needs to make I/O. then it will use bitmap scan and make sequential I/O for all this TIDs.


Observations :

-> Notice that bloom filters can be used only for equality query. you can't utilize them for range queries etc. 

-> Bloom Index can be created on composite columns like B+ Tree index. but Hash index can only be created on single column.

-> Bloom index takes very tiny space. like for 1 M rows, if each row has 128 bit bloom array = 16 byte array, for entire table, it will take 16 MB space. while if you create multi column B+ Tree index, the tree size will be very tall and will take much more size then 16 MB. if you create Hash index, then you will need to create hash index for each required column. which combined will take much more space and will add overhead of maintaining those indexes. 

-> When you create new tuple, DBMS will create its signature and will append it to bloom index. similarly for delete, it will remove that signature. and for update, if you update the columns on which bloom index is build, then it will delete old one and insert new one.


-> So bloom Index requires sequential scan in bloom index file, but due to it's tiny space and very low overhead of maintaining it, it is much more feasible to use for multiple columns. note that for single column, it is not good idea. hash / B+ Tree index will be good for single column. but for multiple columns equality check, bloom index are always perfect choice.