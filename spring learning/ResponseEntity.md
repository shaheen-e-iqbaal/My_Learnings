
it is used to sent more custom http response. it helps to add more configuration to http response.
we can add status code, body, header in response.

Syntax: 

public ResponseEntity<Datatype of Body> getUser(@RequestParam int userId){
      return ResponseEntity.status(HttpStatus.valueOf(200)).
      header("header to be sent").
      body("body to be sent");
}

we can use it with @RestController also.


@ResponseBody is used when we want to send only body and doesn't want to customize the header and status code of the response.
and ResponseEntity is used when we want to sent more customized HTTP response i.e. custom header and status.