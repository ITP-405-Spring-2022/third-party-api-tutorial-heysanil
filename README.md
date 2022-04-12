# ITP 405: API Tutorial

This tutorial will cover the basics of using the [file.io API](https://www.file.io/developers). This API allows you to easily upload, manage, and share files of any kind.

The API can be used for free up to 1 request per minute, or supports paid plans for higher-capacity usage.

Full docs are available at [file.io/developers](https://www.file.io/developers).

## Authorization

The API supports two methods of authentication for each request: HTTP Basic auth and Bearer auth tokens.

API tokens can be retrieved upon signing up for a free [file.io](https://www.file.io/) account.

Both methods only require an API token; with Basic auth, you can provide your API token as a username and omit password. Or, you can just pass your API token as the Bearer token in your request.

## Using the API

The base URL for all requests is `https://file.io`.

There are two main endpoints for the API: `/` and `/{key}`. Requests to `/` are used when there is no specific existing file that the request relates to; for example, searching for files or creating a file (since you won't have an identifier for the file until after creation). Requests to `/{key}` allow retrieval and modification of existing files.

A third endpoint, `/me`, supports `GET` requests and allows you to retrieve account details for the authenticated user (whose token you've provided with the request). This is useful if you're building an app that integrates file.io services, and need to verify the user's API limits before offering them services.

All requests should contain an `Accept` header specifying `application/json` as the expected media type.

## Example requests

We will walk through two example situations: uploading a file, and then later deleting that file. You can refer to the [full API documentation](https://www.file.io/developers) for more details and other available requests.

You can combine these two examples to create a simple file sharing service in PHP powered by file.io's API; users can upload files (`POST /`) and then delete the file in the future whenever they'd like (`DELETE /{key}`).

### Uploading a file

To upload a file, you must use a `POST` request to the `/` endpoint (meaning that your HTTP request should look like `POST https://file.io/`). The request body should be encoded as `multipart/form-data`.

You can provide a few options in the request body:

- `file`: a string or binary stream of the file contents _(required)_
- `expires`: expiration date
- `maxDownloads`: an integer defining the number of times the file can be downloaded before deletion
- `autoDelete`: a boolean defining whether the file should be automatically deleted after it's been downloaded

The API responds in JSON; here is an example response:

```json
{
    "success": true,
    "status": 0,
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "key": "string",
    "name": "string",
    "link": "string",
    "expires": "2022-04-12T21:59:02.949Z",
    "expiry": "string",
    "downloads": 0,
    "maxDownloads": 0,
    "autoDelete": true,
    "size": 0,
    "mimeType": "string",
    "created": "2022-04-12T21:59:02.949Z",
    "modified": "2022-04-12T21:59:02.949Z"
}
```

The `id` should be stored in your database for future operations on this resource. The `link` attribute of the response can be provided to users as the download location for the given file; however, it is generally good practice to just store the file `id` and use a `GET` request to retrieve the download link whenever needed, to ensure that you are always providing an active/valid link.

Here is an example request to this API with Laravel's `Http` facade. Note the usage of `withToken()` and `acceptJson()`; `withToken('{token}')` adds `{token}` as a Bearer token, and `acceptJson()` automatically adds an `Accept: application/json` header.

```php
use Illuminate\Support\Facades\Http;

$response = Http::withToken('{{ your API token }}')
->acceptJson()
->attach(
    'file', file_get_contents('example.jpg'), 'example.jpg',
)->post('https://file.io', [
    // Other options can go here
    'maxDownloads' => '5',
    'autoDelete' => 'true',
]);
```
### Deleting a file

Once a file has been created/uploaded, you will receive an `id` in the response that can later be used for any of the `/{key}` endpoints.

The `DELETE` endpoint is simple, and just requires the same headers as above along with the file key/ID as a parameter.

The server provides a simple repsonse indicating success:

```json
{
    "success": true,
    "status": 0
}
```

Here's an example of how you would do this with Laravel's HTTP client:

```php
use Illuminate\Support\Facades\Http;

$response = Http::withToken('{{ your API token }}')
->acceptJson()
->delete('https://file.io/3fa85f64-5717-4562-b3fc-2c963f66afa6');
```
