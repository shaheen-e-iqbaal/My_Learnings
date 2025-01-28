

Serialization is needed in cases where we need to :

1. Store object in a file
2. Send it over a network


-> to make any object serializable, we need to implement marker interface (A **marker interface** in Java is an interface that contains no methods or fields—it is empty. Its primary purpose is to provide **metadata or a tagging mechanism** to the classes that implement it.) in class of that object.

-> The default java serialization converts objects into a binary format which can't be read by human.

-> to transfer objects over a network, we serialize object in a format like JSON, XML etc.

-> field marked as static and transient are not serialized. 

-> If an object contains references to other objects, those objects are also serialized **recursively**, provided they implement `Serializable`.

-> Fields of the class’s **non-`Serializable` superclasses** are not serialized directly.
- However:
    - If the superclass is `Serializable`, its fields are serialized.
    - If the superclass is not `Serializable`, its state must be restored manually.
