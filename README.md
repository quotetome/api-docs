# QuoteToMe REST API v1

This document describes the QuoteToMe REST API v1.

Contact: support@quotetome.com

## API Access

The QuoteToMe API Server serves the API for the QuoteToMe web and mobile applications. API endpoints generally take the following form:

`/api/v1/SUBJECT/.../`

### Authentication

Each request to the API must be authenticated as a registered QuoteToMe user. 

Send an `Authorization` HTTP header with each request. The header must take the following form:

`Authorization: Token AUTH_TOKEN`

where `AUTH_TOKEN` is the user's authorization token, obtained after performing a successful login.

### Other Header Values

The only header values you should need to send are:

* `Authorization: Token AUTH_TOKEN`
* `Content-Type` , if you are sending data

No other headers should be included or needed.

## Data Formats

The QuoteToMe API exchanges data in the [JSON](https://www.json.org/json-en.html) format. Requests and responses must include a `Content-Type: application/json` HTTP header.

### Request Data

Supply a single JSON object as the HTTP payload for requests that require input. 

The shape and requirements for this JSON object vary depending on the requested operation. Refer to the endpoint's API documentation for complete details.

Endpoints that return a list of objects support pagination. Provide `limit` and `offset` query parameters to the request to have the results paginated. The `offset` parameter specifies how many objects to "skip" and the `limit` parameter specifies how many objects to return, starting from the `offset`.

### Response Data

API responses will always contain a single JSON object even if there is no return data. The general form of this JSON object is:

```json
{
    "status": "string-based status code",
    "message": "an error message",
    "count": 500,
    "next": 60,
    "previous": 20,
    "data": {}
}
```

Each possible field of this object is outlined below.

_**status**_ 

_Present_: Always

Indicates whether the requested operation was completed successfully. Possible values are `success`, `fail`, or `error`. See below for more detail.

_**message**_ 

_Present_: When `status` is `error`

A human-readable error message describing the reason for the failure.

_**data**_ 

_Present_: Whenever the API has a data payload to return

This is the data requested by the caller. If the endpoint returns a single object then this field will contain it's JSON representation. If the endpoint returns a list of objects then this field will contain a list of JSON objects.

_**count**_

_Present_: When result `data` is a list
 
When the results _are not_ paginated this count is the number of results returned. It matches the length of the list in the `data` field.

When the results _are_ paginated this count is the number of object that the API _could_ return if it were to return all of them. If this count is higher than the length of the list in the `data` field then there are more objects available than were returned by this API call.

_**next**_ and _**previous**_

_Present_: When result `data` is paginated

The _offset_ of the next and previous page of results.

### Status Codes

The QuoteToMe API makes use of both HTTP status codes and a special `status` response field to indicate whether or not the operation completed successfully. The details of each are listed below.

#### HTTP Status

The HTTP status code is used to generally indicate the success of an operation.

The following HTTP status codes are returned by the API:

- `2xx` - The operation completed successfully. This is typically a `200` code but others in the 200-range may also be used, such as `201` to indicate that the returned object was newly created.
- `400` - The operation could not be processed. The client must inspect the response data to learn more about the reason for the failure.
- `403` - The client is not allowed to perform this operation. This could be due to user permissions or an invalid auth token.
- `404` - The requested object could not be delivered. This doesn't necessarily indicate that the object doesn't exist, only that the API is refusing to provide it to the client.
- `500` - The API server encountered an internal error that is not the fault of the caller.

> **Note**: Responses with HTTP status `404` and `500` **do not** contain the standard JSON payload in the response.

#### API Status

The `status` field of the API response provides more detail about the state of the operation. Possible values include `success`, `fail`, or `error`. Each is described in detail below

##### success

The operation completed without error. Requested data, if any, are available in the `data` field. 

The `message` field is omitted for these successful operations. The `count`, `next` and `previous` fields will only be included if the response has data output and that output is a list of objects.

A successful response with no data looks like this:

```json
{
  "status": "success"
}
```

A success response with a single data object:

```json
{
  "status": "success",
  "data": {}
}
```

A success response with an unpaginated list of results:

```json
{
  "status": "success",
  "count": 31,
  "data": []
}
```

A success response with a paginated list of results:

```json
{
  "status": "success",
  "count": 214,
  "next": 80,
  "previous": 40,
  "data": []
}
```

##### fail

The operation could not be processed because of invalid input data or other failed validations. 

The `data` object may contain several keys. Each key represents a problem with one of the fields of the input payload. A special key, called `non_field_errors` lists any problems with the operation that aren't related to a single field.

The following is an example of a failure data payload:

```json
{
  "status": "fail",
  "data": {
    "non_field_errors": [
      "Warning: Your account is awaiting verification."
    ],
    "name": "Name must not contain any spaces"
  }
}
```

A typical usage scenario for this error data is to display the `non_field_errors` as a general error notification and use the other keys to put form fields into an error state with the given error message.

##### error

The operation could not be completed due to some processing error unrelated to input validation. The `message` field will be present and contain a short human-readable explanation of the failure.

This is an example of an error response:

```json
{
    "status": "error",
    "message": "The QuoteToMe system is currently under maintenance."
}
```

There may also be a `data` field. If present it contains information useful for troubleshooting the error, but should not be made visible to the user.
