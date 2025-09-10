
The top challenges during the data transferring are : Data integrity, Data confidentiality, Authenticated data source etc.


**Encryption :**

It is a process of encrypting raw data to some garbage type data which can't be understood by any human. we can decrypt (get back raw data) using proper key.

So, the main purpose of the encryption is to achieve the data confidentiality during the transfer of data.

it has mainly two types :  1. Symmetric 2. Asymmetric


Symmetric :  in this encryption we use same key to encrypt and decrypt the data. this key is only available to both sender and receiver. if this key get's leaked, then anyone can decrypt the data. 

so major challenge of symmetric method is to securely transfer the private key of receiver to sender. one way to do this is to transfer the secret key using asymmetric method.

Ex : AES, DES uses Symmetric encryption.


Asymmetric : in this method, the raw data is encrypted using the receivers public key and then receiver can decrypt it using it's private key. so for both encryption and decryption, only receivers keys are used.

so no middle man can decrypt the data, because it doesn't have the private key of receiver.

so the desired receiver can only decrypt the data.

Ex: RSA, deffie halman


In symmetric, the key is small and operations to encrypt and decrypt are also like xor, multiply and on the other side in asymmetric, keys are large and operation are also like power etc. 
so due to this fact symmetric encrypt/decrypt are faster than asymmetric encrypt/decrypt.

so the common way is to first transfer the secret key between sender and receiver using asymmetric method, then switch to symmetric method for future encrypt/decrypt operation.


**Hashing:**

it is way to convert raw data of any length into fixed size of string called hash value or hash code. 

so one of the property of hash algorithms is that they will produce the fixed length of hash value for any length of input text.

we can't convert the hash value to it's original form. means it is only one way encryption.

Hashing is used to check data integrity throughout the data transmission.

Usage of hashing is to store the password in database, in digital signature to check the received data integrity.

Ex: MD5, SHA256


**Encoding:**

**Encoding** is the process of converting data or information from one format to another, typically for the purpose of transmission, storage, or processing by a computer. The original data is transformed into a standardized representation that can be easily interpreted or transmitted by machines or systems.

ex : converting normal text into UTF-8, ASCII.

we can get back the encoded data to it's original form also.



**Digital Signature:**

It is used to (authenticate the sender + data integrity) of the data. 

Flow:  

Raw Data -> Hash function -> hash value -> encrypt using ==**private key of sender**== (call it Packet).

Message to send = ==Raw data + Packet==.


so sender sends the both packet + Raw Data. Receiver than tries  to ==decrypt the packet using public key of sender== and if it does it successfully, then it is sure that the data is sent from the authenticate / desired sender. 
after decrypting it, it generates the hash value of raw data it got and compares that hash value with the decrypted one. if they are equal, it means that raw data is not tempered during transmission otherwise it is tempered.


so using digital signature, we can ensure two thing : 1. Source is Authenticate. 2. Data Integrity.


if we also want to ensure the Data Privacy / confidentiality, than what we can do is, 
we can use asymmetric encryption for raw data instead of sending the raw data itself. we first encrypt it using the public key of receiver, than sent it. so receiver will decrypt it using it's own private key. 

so using above method + digital signature, we can get Data confidentiality, Data integrity, Sender Authentication.







