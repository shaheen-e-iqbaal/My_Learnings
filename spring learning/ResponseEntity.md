
it is used to sent more custom http response. it helps to add more configuration to http response.
we can add status code, body, header in response.

Syntax: 

`public ResponseEntity<Datatype of Body> getUser(@RequestParam int userId){
      `return ResponseEntity.status(HttpStatus.valueOf(200)).
      `header("header to be sent").
      `body("body to be sent");`
`}`

we can use it with @RestController also.


@ResponseBody is used when we want to send only body and doesn't want to customize the header and status code of the response.
and ResponseEntity is used when we want to sent more customized HTTP response i.e. custom header and status.

-> inside @RestController, if we return normal POJO or string and don't use ResponseEntity, spring will wrap our response around ResponseEntity with body as return value, status code as default 200.

Status Codes :

200 : OK : GET Call. request is successful and we returned response body.
201 : Created : POST Call : request is successful and new data is added.
204 : No Content : DELETE Call : request is successful and we are not returning anything in body.
206 : Partial Content : POST Call : request is partial success. like for 100 object insertion request, we added 95 and 5 insertion failed.

3xx : Client need to redirect

400 : Bad Request : Client is not passing required info to process request.

401 : Unauthorized : Client is not passing authentication token, or related details which are required to access that API endpoint.

403 : Forbidden : Client isn't authorized to access that resource

404 : Not Found : the requested resource by client is not found. like client wants to get User with ID = 123, and no user is present in DB with this id.

5xx : Server side issue is there.