
-> at high level, we can say database represent something like real world. so it should always be logically correct. like age of something always be non negative.

-> It means that database should be in logically correct state after the transaction if it was in correct state before start of the transaction.

-> You can define integrity constraints on columns like age shouldn't be negative... to ensure that your entity remain consistent with real word.

-> There are two types of consistency 1. Strong consistency or consistency 2. eventual consistency.

-> When we have single database that is running on multiple machines (i.e. distributed database), if we modify some data in database running in first machine, and immediately check it in other databases running on different machine, should that change be visible or not.
-> in strong consistent or consistent database, it should be visible, but in eventual consistent database, it will be visible after sometime i.e. eventually.
