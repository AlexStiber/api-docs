# Generating Content
The content generation through the API is accessible when all mandatory information is present in the object.

## Generate content for a single Object
```python
import axsemantics
axsemantics.login('', '')

cp = axsemantics.ContentProject.all().get(id=1)
thing = cp.things().get(id=334)
response = thing.generate_content(force=False)
```

```shell
$ curl --request POST \
  --url "https://api.ax-semantics.com/v1/content-project/1/thing/123/generate_content/?force=true" \
  --header 'Authorization: Token aa5d2e36668c11e5964038bc572ec103'
```

> The response contains informations regarding the content request, such as:

```json
{ "status": "CALLED",
  "number": 1,
  "content_request": {
    "id": 123,
    "content_project": 1,
    "generated_text_in_html": null,
    "thing_type": "demo",
    "state": "Started",
    "created": "2015-10-02T11:56:45.994940Z",
    "modified": "2015-10-02T11:56:46.068082Z",
    "...":"..." }}
```

### Endpoint
`POST /v1/content-project/{CP_ID}/thing/{OBJ_ID}/generate_content/{force}`

In the example you have to exchange `{CP_ID}` with a valid Content Project ID and `{OBJ_ID}` with a valid Object ID. *Keep in mind that this is not the UID but the object ID assigned by the platform!*.

`{force}` is a query parameter which is used to define whether the Content Request should discard and regenerate existing content. This parameter is optional, the default is **force=false**.

- **?force=false** (default): Content will be requested if this object has no existing content. Otherwise no action is taken.
- - **?force=true**: Existing content will be discarded and regenerated.

## Generate content for a whole Content Project
```python
import axsemantics
axsemantics.login('', '')

cp = axsemantics.ContentProject.all().get(id=1)
response = cp.generate_content(force=False)
```

```shell
$ curl --request POST \
  --url "https://api.ax-semantics.com/v1/content-project/2123/generate_content/?force=true" \
  --header "Authorization: Token aa5d2e36668c11e5964038bc572ec103"
```

> The API responds by returning a JSON response, such as:

```json
{"status":"CALLED","number":3}
```
### Endpoint
`POST /v1/content_project/{CP_ID}/generate_content/{force}`

In the example you have to exchange `{CP_ID}` with a valid Content Project ID.

`{force}` is a query parameter which is used to define whether the content request should discard and regenerate existing content. This parameter is optional, the default is **force=false**.

## Request status report for a content request
```python
import axsemantics
axsemantics.login('', '')

cp = axsemantics.ContentProject.all().get(id=1)
thing = cp.things().get(id=334)
print(thing['status'])
```

```shell
$ curl --request GET \
    --header "Authorization: Token aa5d2e36668c11e5964038bc572ec103" \
    --url "https://api.ax-semantics.com/v1/content-project/1/thing/123/"
```

> The API response includes a status field:

```json
{ "id": 123,
  "uid": "demo",
  "status": "not requested",
  "generated_text": null,
  "content_project": 1,
  "...":"..." }
```

### Endpoint
`GET /v1/content-project/{CP_ID}/thing/{OBJ_ID}/`

In the example you have to exchange `{CP_ID}` with a valid Content Project ID and `{OBJ_ID}` with a valid Object ID. *Keep in mind that this is not the UID but the ID assigned by the platform!*.

## Export generated content for a single Object
```python
import axsemantics
axsemantics.login('', '')

cp = axsemantics.ContentProject.all().get(id=1)
thing = cp.things().get(id=334)
print(thing['generated_text'])
print(thing['text_as_html'])
```

```shell
$ curl --request GET \
  --url https://api.ax-semantics.com/v1/content-project/1/thing/123/ \
  --header 'Authorization: Token aa5d2e36668c11e5964038bc572ec103'
```

> The API returns information regarding the Object, including generated text if any exists:

```json
{ "id": 456,
  "content_project": 1,
  "state": "Success",
  "generated_text_in_html": "<h1>Überschrift</h1>\n<p>Absatz</p>",
  "generated_text": "# Überschrift\nAbsatz",
  "...": "..." }
```

The generated content is available in its original format or in HTML-format. Usually the original format is [Markdown](https://daringfireball.net/projects/markdown/syntax/).

### Endpoint
`GET /v1/content-project/{CP_ID}/thing/{OBJ_ID}/content_request/`

In the example you have to exchange `{CP_ID}` with a valid Content Project ID and `{OBJ_ID}` with a valid Object ID. *Keep in mind that this is not the UID but the ID assigned by the platform!*.

## Push for new content via web hooks

On request, we can activate the push feature for new content: When a new text is generated MyAX will send a HTTP POST request to the web hook URL.

It has a signature header to verify the integrity/authenticity and some data about the object and the text in the post data body.

### Signature header

```
HTTP_X_MYAX_SIGNATURE: "sha1=df589122eac0f6a7bd8795436e692e3675cadc3b"
```

The checksum is calculated as hmac sha1 hexdigest. The key is your API token. The message is the post data.

### POST data

You will receive POST data looking like this:

```
{
  "id": 9001,
  "name": "<The name of the object>",
  "text": "<The new text (raw)>",
  "text_as_html": "<The new text (as html from markdown)>",
  "uid": "<The uid you put in the object>",
  "content_project_name": "<Name of the content project>",
  "content_project_id": 1337,
  "text_modified": "2015-10-21T23:29:00.000000+00:00",
  "language": "en-US"
}
```

Here is an example how you can verify the encryption 'hmac' on your side: [hmac example gist](https://gist.github.com/mfa/f12fadd6b707127de3338d673c839e4d)



## Export generated content for an entire Content Project

If you want to have the content available as one big download file, you can use the export functionality.

```shell
$ curl --request GET \
  --header 'Authorization: Token aa5d2e36668c11e5964038bc572ec103' \
  --url 'https://api.ax-semantics.com/v1/download-exports/?page=1&page_size=10'
  # yields a list of available downloads
  # find your required download and use its 'download_url' attribute
$ curl --request GET \
  --output export.xlsx \
  --url https://api.ax-semantics.com/v1/content_project_export_download/7f9cc6a2-6b55-11e5-bb84-5e2c2d9baef2
```
### Endpoint
`GET /v1/download-exports/`

