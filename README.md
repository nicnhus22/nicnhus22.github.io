# API Design Guidelines

These design guidelines must be referred to whenever a new API is being designed (and developed). The content is intended to 


# HTTP Protocol

REST APIs are designed around the rich HTTP protocol.

## HTTP Verbs

> **Normalize HTTP verbs usage to keep the API intuitive. Only the 5 verbs below should be used : use the right verb for the right operation to perform** 

|  Method | Action |
|--|--|
|  `GET` | Method **requests data** from the resource and should not produce any side effect |
|  `POST` | Method requests the server to **create** a resource in the database |
|  `PUT` | Method requests the server to **update** resource or **create** the resource, if it doesn’t exist |
|  `PATCH` | Method requests the server to **partially update** resource |
|  `DELETE` | Method requests that the resources, or its instance, **should be removed** from the database|


## Idempotent vs. Safe 

> **Keep idempotent and safe principles in mind when developing APIs. HTTP verbs that are idempotent must not lead to non idempotent operations and vice versa.** 


### Idempotent

An idempotent HTTP method is a HTTP method that **can be called many times without different outcomes**. It would not matter if the method is called only once, or ten times over. **The result should be the same**. 

**Example**

    customer.age = 20 // idempotent

> This example  is **idempotent** : no matter how many times we execute this statement, the customer’s age will always be set to 20. 

    customer.age += 1 // non-idempotent

> The second line **isn't idempotent** : executing this 10 times will result
    in a different outcome as when running 5 times.
 
### Safe

Safe methods are HTTP **methods that do not modify resources**. Meaning if you use a safe HTTP method it should never have an create, update or delete resources. In addition, safe methods are methods that can be cached, prefetched without any repercussions to the resource.

**Example** 

    GET https://dev.api.inetpsa.com/project/customers/delete HTTP/1.1

> The following example is incorrect if this API call actually deletes the resource
> Notice the `GET` method and the `/delete` in the URI

### Summary Table

|               |Idempotent     |Safe       |
|---------------|-------------- |-----------|
|`GET`			|**Yes**		|**Yes**		|
|`POST`			|No			|No					|
|`PUT`			|**Yes**			|No					|
|`PATCH`			|No						|No			|
|`DELETE`			|**Yes**			|No					|

## HTTP Status Codes

> **HTTP status codes are used to indicate to the client the state of a request (success, error, information etc.) and bring an additional functional information. Refer to this list whenever you want to return responses. Limit to the codes below.** 

| Code|Meaning       |Meaning
|-----|-------------|-------------|
|`200`|**OK**|Standard response for successful HTTP requests. The actual response will depend on the request method used. In a GET request, the response will contain an entity corresponding to the requested resource. In a POST request the response will contain an entity describing or containing the result of the action.|		
|`201`|**Created**|The request has been fulfilled and resulted in a new resource being created. Successful creation occurred (via either POST or PUT). Set the Location header to contain a link to the newly-created resource (on POST). Response body content may or may not be present.|		
|`204`|**No Content**|The server successfully processed the request, but is not returning any content. The 204 response MUST NOT include a message-body, and thus is always terminated by the first empty line after the header fields.|		
|`300`|**Not Modified**|Used for conditional GET calls to reduce band-width usage. If used, must set the Date, Content-Location, ETag headers to what they would have been on a regular GET call. There must be no body on the response.|		
|`400`|**Bad Request**|The request could not be understood by the server due to malformed syntax. The client SHOULD NOT repeat the request without modifications. |		
|`401`|**Unauthorized**|Similar to 403 Forbidden, but specifically for use when authentication is possible but has failed or not yet been provided|		
|`403`|**Forbidden**|The server understood the request, but is refusing to fulfill it. It SHOULD describe the reason for the refusal in the entity. If the server does not wish to make this information available to the client, the status code 404 (Not Found) can be used instead|		
|`404`|**Not Found**|Used when the requested resource is not found, whether it doesn't exist or if there was a 401 or 403 that, for security reasons, the service wants to mask|		

# JSON Format

> **APIs must use JSON formatted input and output.**

**Example** 
```json
"user" : {
    "id": 19083974, 				// Number
    "name": "John Doe",				// String
    "email": "john.doe@psa.com",
    "age": 19,
    "roles": ["admin", "editor"], 	// Array
    "disabled" : false,				// Boolean
    "address":{ 					// Object
        "city": "Paris"
	}
}
```

## Object Key

> **A key or attribute must be unique for any given level of data**

 - **Bad example**
	 ```json
	GET /customers/19083974  HTTP/1.1

    "customer" : {
	    "id": 19083974,
	    "name": "John",
	    "name": "Doe",		// Using name twice on the same level of data is forbidden
	} 
	```
 - **Good example**
	 ```json
    GET /customers/19083974  HTTP/1.1
    
    "customer" : {
	    "id": 19083974,
	    "name": "John Doe",
	    "address": {
		    "name": "Home",	// This is allowed since it belongs to address and not customer
		    "city": "Paris"
	    }
	} 
	```

> **Respect the informational context by using clear and explicit naming and use generic terms reusable in a different context than the application it  was first designed for.**

 - **Bad example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "customeId": 19083974, // Bad naming since a customer could be a different object in a different context
	    "home_address" : { 	  // Not developer friendly as other projects might call it differently
		    "city":"Paris"
	    }
	} 
	```
 - **Good example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "id": 19083974,		// Good generic naming : can be used in any context
	    "address": {			// Good generic naming
		    "name": "Home",	// Better naming as
	    }
	} 
	```

## Object Value

> **JSON is a loosely typed format.  Nonetheless, formats must be  standardize for any JSON objects (see payload conventions)**



> **Null objects must never be included in a JSON response because it decreases payload size and facilitates API/resource evolution**

 - **Bad example**
	 ```json
	GET /customers/19083974  HTTP/1.1
	
    "customer" : {
	    "id": 19083974, 
	    "address" : null	// Do not return if null
	} 
	```
 - **Good example**
	 ```json
	 GET /customers/19083974  HTTP/1.1
	 
    "customer" : {
	    "id": 19083974,		// Good generic naming : can be used in any context
	} 
	```
