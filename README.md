# go-api
In this code, APIs are created with Go. 

Dependency used is gin. 

Example of command to run app:

```
go run main.go 
```

Example of command to get response from API:

####GET 
```shell
curl localhost:8080/books/2
```

####POST
```shell
curl localhost:8080/books --include --header "Content-Type:application/json" -d @body.json --request "POST"
```
-d is for data. @ is for file. 

####PATCH
```shell
curl localhost:8080/checkout?id=2 --request "PATCH"
```

###Difference between GetQuery and Param :

####GetQuery
```go
id, ok := c.GetQuery("id")
```
sending request:
```shell
curl localhost:8080/checkout?id=2 --request "PATCH"
```
####Param
```go
id := c.Param("id")
```
sending request:
```shell
curl localhost:8080/books/2
```

###Difference between Patch and Put :
I had a question of why we weren't using Put instead of Patch method in the case of checkout or return book where we 
update the quantity.

The main difference between PUT and PATCH in REST API is that PUT handles updates by replacing the entire entity, while PATCH only updates the fields that you give it.

PATCH does not change any of the other values.

If you use the PUT method, then everything will get updated. Depends on server implementation, but in most REST APIs, this means it will overwrite any missing fields to null.

PATCH method updates individual fields without overwriting existing fields.


### Explaining methods used in the code:
```go
package main 

func getBooks(c *gin.Context) {
	c.IndentedJSON(http.StatusOK, books) // c.JSON() preferred (see below)
}

// Example invalid function not in code just to show use of gin.H
func errorMessage(err error) {
	if err != nil {
		c.IndentedJSON(http.StatusNotFound, gin.H{"message": err.Error()})
		return
	}
}

func main() {
    router := gin.Default()
    router.GET("/books", getBooks)
    router.Run("localhost:8080")
}
```

**gin.Context**  

gin context stores all the information of the request to send response

**c.IndentedJSON** 

IndentedJSON serializes the given struct as pretty JSON (indented + endlines) into the response body. It also sets the Content-Type as "application/json". 

WARNING: we recommend using this only for development purposes since printing pretty JSON is more CPU and bandwidth consuming. Use Context.JSON() instead i.e. c.JSON()

**gin.Default()** 

Default returns an Engine instance with the Logger and Recovery middleware already attached.

**gin.H{}**

gin.H writes own custom json like the message param above. maps string type key to interface type value

###Binding in gin

```go
func createBook(c *gin.Context) {
	var newBook book

	if err := c.BindJSON(&newBook); err != nil {
		return
		// returning does not automatically return a response
		// The BindJSON() method is what will handle sending the error response
	}

	books = append(books, newBook)
	c.IndentedJSON(http.StatusCreated, newBook)
}
```

####Type - Must bind

Methods - Bind, BindJSON, BindXML, BindQuery, BindYAML, BindHeader, BindTOML

Behavior - These methods use MustBindWith under the hood. If there is a binding error, the request is aborted with c.AbortWithError(400, err).SetType(ErrorTypeBind). This sets the response status code to 400 and the Content-Type header is set to text/plain; charset=utf-8. Note that if you try to set the response code after this, it will result in a warning [GIN-debug] 

[WARNING] Headers were already written. Wanted to override status code 400 with 422. If you wish to have greater control over the behavior, consider using the ShouldBind equivalent method.

**Type - Should bind**

Methods - ShouldBind, ShouldBindJSON, ShouldBindXML, ShouldBindQuery, ShouldBindYAML, ShouldBindHeader, ShouldBindTOML,

Behavior - These methods use ShouldBindWith under the hood. If there is a binding error, the error is returned and it is the developer's responsibility to handle the request and error appropriately.
When using the Bind-method, Gin tries to infer the binder depending on the Content-Type header. If you are sure what you are binding, you can use MustBindWith or ShouldBindWith.

You can also specify that specific fields are required. If a field is decorated with binding:"required" and has a empty value when binding, an error will be returned.
