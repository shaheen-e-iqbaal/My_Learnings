
There are several ways to store password in DB.

1. First Encrypt using any key and then store.
2. Generate Hash using algorithms like MD5, SHA1,SHA526 etc. and then store this hash value.
3. add salt to password and then follow the step 2.


1. It is a two way i.e. we can encrypt the password using any encryption key and then store it in database. and we can also decrypt the encrypted password to get original password. 

   This method is not secure because if the encryption key and database  gets leaked, then anyone can decrypt the encrypted passwords present in database.


2. in this method, we generate the hash value of password using algorithm like MD5 etc. and store this hash value in database. so during authentication, when user gives us the password, we generates its hash value and check for it inside database.

   It is a good approach then first, but it is vulnerable for brute force attack.

3. in this, we keep one string called salt. we add this string to the password and then generate hash value. so it becomes little more secure then second approach.