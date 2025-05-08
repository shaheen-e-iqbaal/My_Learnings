

-> fastapi can automatically serialise python objects like dictionary, list, tuple. if you return them directly from endpoint, they will be converted into JSON by default by fastapi.

-> pydantic models can also automatically serialised by fastapi.

-> Custom python objects like user of User class, are not by default serialisable. if we return them directly from endpoint method, fastapi will throw value error. if our return type is custom python object which is not serialisable, we have to convert it into dictionary and then return that dictionary.
		ex : def to_dict(self):
		       return {
	              "id": self.id,
	              "name": self.name,
	              "email": self.email
               }

-> We don't need to convert our response into json, we just have to make sure that our return type is serialisable. conversion to json will be taken care by fastapi. if we serialise our response into json before returning, then fastapi will also serialise that already serialised response into json, which will be two times serialisation, so response won't look like a desired one.



-> to use fastapi automatic json parsing, either return pydantic model corresponding to user object or return dictionary representation of user object.



**Difference between json.dumps(obj) and json.dump(obj, fp) method :**

-> json.dumps method returns json string for obj object. obj must be serialisable object.

-> json.dump(obj, fp) will convert obj into json string and will write it to file fp.



***Sending Custom response with headers, body and status code***

-> We can use Response(content = any_string, headers = some_headers, status_code = 201)
	here datatype of content must be a string, if we want to return user object, we can convert it into json string like content = json.dumps(user.to_dict()). note that we have passed dictionary to dumps method, because it expect serialisable object and our user in python object which is not serialisable.
