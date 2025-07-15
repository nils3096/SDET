# Velocity Template Language (VTL) Storage Guide

## The `#set` Directive

The syntax `#set($tc.saved.serviceQualificationRequest = "JSON response stored")` is Velocity Template Language (VTL) syntax used to store values in variables.

## Basic Syntax

```velocity
#set($variable = value)
```

### Examples of Storage:

#### 1. Simple String Storage
```velocity
#set($message = "Hello World")
#set($tc.saved.serviceQualificationRequest = "JSON response stored")
```

#### 2. Numeric Values
```velocity
#set($count = 42)
#set($price = 19.99)
```

#### 3. Complex Object Storage
```velocity
#set($tc.saved.userInfo = {
    "name": "John Doe",
    "email": "john@example.com",
    "id": 123
})
```

#### 4. Storing API Response Data
```velocity
## In Postman or API testing tools:
#set($tc.saved.responseData = $response.body)
#set($tc.saved.authToken = $response.headers.authorization)
#set($tc.saved.userId = $response.body.user.id)
```

## Common Use Cases

### 1. **Postman Testing**
In Postman pre-request or test scripts:
```javascript
// JavaScript equivalent in Postman:
pm.test.replaceIn("#set($tc.saved.serviceQualificationRequest = \"{{responseBody}}\")")
```

### 2. **API Response Storage**
```velocity
## Store entire response
#set($tc.saved.fullResponse = $response)

## Store specific fields
#set($tc.saved.requestId = $response.body.requestId)
#set($tc.saved.status = $response.body.status)
```

### 3. **Template Variable Storage**
```velocity
## Store for later use in templates
#set($tc.saved.baseUrl = "https://api.example.com")
#set($tc.saved.apiVersion = "v1")
#set($tc.saved.endpoint = "$tc.saved.baseUrl/$tc.saved.apiVersion/users")
```

## Accessing Stored Values

After storing with `#set`, access the values using:

```velocity
## Access the stored value
$tc.saved.serviceQualificationRequest

## Use in conditions
#if($tc.saved.serviceQualificationRequest == "JSON response stored")
    ## Do something
#end

## Use in other assignments
#set($newVariable = $tc.saved.serviceQualificationRequest)
```

## Best Practices

1. **Naming Convention**: Use descriptive names
   ```velocity
   #set($tc.saved.userAuthToken = $authToken)
   #set($tc.saved.lastApiCallTimestamp = $currentTime)
   ```

2. **Null Checking**: Always check if values exist
   ```velocity
   #if($tc.saved.serviceQualificationRequest)
       ## Value exists, use it
       $tc.saved.serviceQualificationRequest
   #else
       ## Default value or error handling
       "No data stored"
   #end
   ```

3. **Type Safety**: Be aware of data types
   ```velocity
   ## String storage
   #set($tc.saved.stringValue = "text")
   
   ## Number storage
   #set($tc.saved.numberValue = 123)
   
   ## Boolean storage
   #set($tc.saved.booleanValue = true)
   ```

## Context-Specific Usage

### Postman/Newman
```javascript
// In Tests tab:
pm.test("Store response data", function () {
    const responseJson = pm.response.json();
    pm.test.replaceIn("#set($tc.saved.serviceQualificationRequest = \"" + JSON.stringify(responseJson) + "\")");
});
```

### Apache Velocity Templates
```velocity
## In .vm template files
#set($tc.saved.pageTitle = "Service Qualification")
#set($tc.saved.currentUser = $user.getName())

<title>$tc.saved.pageTitle</title>
<p>Welcome, $tc.saved.currentUser!</p>
```

### Testing Frameworks
```velocity
## Store test data
#set($tc.saved.testData = {
    "expectedStatus": "success",
    "expectedCode": 200,
    "expectedMessage": "Operation completed"
})

## Use in assertions
#if($response.status == $tc.saved.testData.expectedStatus)
    ## Test passed
#end
```

## Your Specific Example

```velocity
#set($tc.saved.serviceQualificationRequest = "JSON response stored")
```

This line:
- Creates/updates a variable in the `$tc.saved` namespace
- Stores the string `"JSON response stored"` 
- Can be accessed later as `$tc.saved.serviceQualificationRequest`

To use the stored value:
```velocity
## Display the value
Response status: $tc.saved.serviceQualificationRequest

## Use in conditions
#if($tc.saved.serviceQualificationRequest == "JSON response stored")
    The service qualification request was processed successfully.
#end
```