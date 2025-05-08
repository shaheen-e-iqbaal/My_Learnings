

-> Path variables are parts of the URL path that are used to identify a specific resource or entity. They are dynamic values embedded directly in the URL structure. so when we write API, we have include path variables also in it.

ex : URL = `users/123/posts/21`   API : `/users/{userId}/posts/{postId}`


-> Request parameters are key-value pairs appended to the URL after a `?`. They are used to filter, sort, or provide additional options for the request. so, when we write API, they are not included in it.

ex : URL : `items/search?q=apple&category=fruits` API : `items/search`

#### **1. Path Variables**

- **What are they?**  
    Dynamic values embedded in the URL path to identify a specific resource.
    
- **Syntax:**  
    `/users/{userId}/posts/{postId}`
    
    - `{userId}` and `{postId}` are path variables.
    - API for above URL = `/users/{userId}/posts/{postId}
        
- **When to use:**
    
    - To uniquely identify a resource (e.g., user, product, post).
        
    - When the variable is mandatory for the request.
        
    - For hierarchical URL structures.
    
- **Example:**
    
    GET /users/123/posts/456
    
    - `userId = 123`, `postId = 456`.
    

---

#### **2. Request Parameters (Query Parameters)**

- **What are they?**  
    Key-value pairs appended to the URL after `?` to filter, sort, or provide additional options.
    
- **Syntax:**  
    `/search?q=apple&category=fruits`
    
    - `q` and `category` are request parameters.
    - API for above URL = `/search`
    
- **When to use:**
    
    - For optional data (e.g., filters, sorting, pagination).
    - When the data is not part of resource identification.
    - For non-hierarchical data.

- **Example:**
    
    GET /search?q=apple&category=fruits&page=2
    
    - `q = apple`, `category = fruits`, `page = 2`.


---

#### **Key Differences**

|Feature|Path Variables|Request Parameters|
|---|---|---|
|**Location in URL**|Part of the URL path|After `?` in the query string|
|**Purpose**|Identify a specific resource|Provide additional data or options|
|**Mandatory/Optional**|Usually mandatory|Usually optional|
|**Example**|`/users/123`|`/search?q=apple&page=2`|

---

#### **Can There Be Multiple Path Variables or Request Parameters?**

- **Path Variables:**  
    Yes. Example: `/users/{userId}/posts/{postId}`
    
    - `userId` and `postId` are both path variables.
    
- **Request Parameters:**  
    Yes. Example: `/search?q=apple&category=fruits&page=2`
    
    - `q`, `category`, and `page` are all request parameters.
    

---

#### **Combining Path Variables and Request Parameters**

You can use both in the same URL:

GET /users/123/posts?status=published&sort=desc

- Path Variables: `123` (userId).

- Request Parameters: `status=published`, `sort=desc`.


---

#### **Handling Encoded URLs**

- URLs are often encoded to handle special characters (e.g., spaces, `&`, `=`).
    
- Spring Boot automatically decodes the URL before processing it.
    
- Example:
    
    - Encoded URL: `/search?q=apple%20pie&category=fruits`
    
    - Decoded URL: `/search?q=apple pie&category=fruits`
    

---

#### **Spring Boot Code Example**


`@GetMapping("/users/{userId}/posts/{postId}")
`public String getUserPost(
        `@PathVariable Long userId,              // Path variable
        `@PathVariable Long postId,              // Path variable
        `@RequestParam(required = false) String status,  // Optional query parameter
        `@RequestParam(defaultValue = "asc") String sort // Query parameter with default value
`) {
    `return String.format("User ID: %d, Post ID: %d, Status: %s, Sort: %s", userId, postId, status, sort);
}`

---

#### **Best Practices**

1. Use **path variables** for resource identification.  
    Example: `/users/123`
    
2. Use **request parameters** for optional or additional data.  
    Example: `/users?status=active&sort=asc`
    
3. Keep URLs clean and meaningful. Avoid overloading them with too many parameters.
    
4. Follow the standard URL structure:
    
    scheme://host:port/path?query#fragment
    

---

#### **Key Takeaways**

- **Path Variables:** Identify resources, mandatory, part of the URL path.
    
- **Request Parameters:** Provide additional data, optional, part of the query string.
    
- **Encoded URLs:** Automatically decoded by Spring Boot.
    
- **Combining Both:** Use path variables for resource identification and request parameters for filtering or sorting.