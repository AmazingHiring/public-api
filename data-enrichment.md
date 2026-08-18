# AmazingHiring Data Enrichment API v1

Data Enrichment API is a standalone service that resolves a single contact point
(email, phone, link or AH profile id) into a full AmazingHiring profile.

It is **not** the same thing as the in-product "Data Enrichment" feature available to
AmazingHiring users in the web UI — this is a separate service with its own tokens,
its own licenses and its own quotas.

Current version: 1.

Base API url: **<https://search.amazinghiring.com/api/dataenrichment/v1/>**

Interactive Swagger UI: **<https://search.amazinghiring.com/api/dataenrichment/docs>**
\(note there is no trailing slash\), machine readable OpenAPI spec:
**<https://search.amazinghiring.com/api/dataenrichment/docs/swagger.json>**. Both are
served by the service itself, so they always describe the version that is running.

## How it works

The API is **asynchronous**. You do not get the profile in the response to your request.

1. You `POST /v1` with one search field. The service answers `201` immediately with a
   request id (`rid`) and status `1 (In Progress)`.
2. The service asks the AmazingHiring backend to find the profile. The backend searches
   (including a real-time crawl) and calls the service back. This normally takes from a
   few seconds up to **~15 minutes**.
3. You get the result either by polling `GET /v1/{rid}`, or — if a callback url is
   configured for your account — by receiving a `POST` to your callback url.

Requests for `profile_id` are handled differently: they are not searched, the profile is
fetched directly from the backend, and the fetch is intentionally **delayed by 15 minutes**
after request creation.

## Authorization

All `/v1/**` endpoints require an access token. `/health`, `/status` and `/docs` are open.

Keep your token **and** your secret private: they act on behalf of your company.

### Access token generation

Tokens are issued by AmazingHiring — contact your
[AmazingHiring sales manager](mailto:sales@amazinghiring.com). Together with the token
you receive a **secret**, which is used to sign requests and to verify responses, and,
optionally, a **callback url** registered for your account.

### Use the access token

The token must be sent in the `Authorization` header, prefixed with the literal `Token`:

```http
Authorization: Token a0b1c2d3e4f5
```

Only the `Token` scheme is supported — this service has no oAuth2.0 support.

| Situation | Response |
| --- | --- |
| `Authorization` header missing | `401 Unauthorized` (empty body) |
| Scheme is not `Token` / header is malformed | `403 Forbidden`, `Invalid authorization scheme` |
| Unknown token | `403 Forbidden`, `Invalid credentials` |

## Request signature

Every `POST`, `PUT` and `PATCH` request body must be signed with your secret and sent in
the `X-Signature` header. An unsigned or wrongly signed request is rejected with
`403 Forbidden` (`Signature is absent or not valid`).

The signature is `PBKDF2-HMAC-SHA256` with **the secret as the password** and
**the request body as the salt**, 2048 iterations, hex-encoded:

```python
import hashlib
import json

body = {"email": "dev@amazinghiring.com"}
signature = hashlib.pbkdf2_hmac(
    'sha256',
    SECRET.encode(),
    json.dumps(body).encode(),
    2048,
).hex()
```

> The service re-serializes the parsed JSON body with Python's `json.dumps` before hashing
> (`", "` / `": "` separators, key order as received). Sign exactly the bytes you send, in
> that canonical form, otherwise the signatures will not match.

**Responses are signed too.** Every response — including empty ones and including the
callbacks the service sends to you — carries an `X-Signature` header computed the same way
over the response body. You are encouraged to verify it.

## Enrichment request statuses

The status of a request is returned as a numeric `code` plus a human readable `message`.

| code | message | meaning |
| --- | --- | --- |
| `0` | `Success` | Profile found, `profile` is filled |
| `1` | `In Progress` | Still being processed, `profile` is `null` |
| `2` | `Not Found` | Processing finished, no profile matched |
| `201` | `Failed` | The backend call failed after all retries |
| `202` | `Timed Out` | Nothing came back within 24 hours |
| `203` | `Limits Exceeded` | A profile was found but your license had no searches left |

## Error codes

Errors are returned as JSON with an application-level `code` field:

| HTTP | code | message |
| --- | --- | --- |
| `400` | `101` | Validation error, see the request schema |
| `404` | `102` | `No request found for '{rid}'` |
| `401` | `103` | `No valid license was found.` |
| `401` | `201` | `Access will start at {date}.` |
| `401` | `202` | `License expired at {date}.` |
| `429` | `203` | `Limits exceeded. No searches left.` |

> Note that error codes `201`, `202`, `203` are a different enumeration from the request
> status codes with the same numbers above. Error codes only ever appear in `4xx`
> responses, in a body that has no `rid`.

`401`/`403` responses produced by the authorization and signature layers have an empty
body and carry the reason in the HTTP status line.

## Enrichment requests

* [**POST** `/v1` Creates enrichment request](#create-enrichment-request)
* [**GET** `/v1/{rid}` Returns enrichment request by id](#get-enrichment-request-by-id)
* [**GET** `/v1/quota` Returns license and quota info](#get-quota)

Trailing slashes are optional: `/v1` and `/v1/`, `/v1/{rid}` and `/v1/{rid}/` are the same
endpoints.

### Create enrichment request

**POST** `/v1`

**Exactly one** of the search fields must be provided:

| field | type | description |
| --- | --- | --- |
| `email` | string | Email in plain text, validated |
| `email_hashed` | string | MD5 hex digest of the email |
| `phone` | integer | Normalized phone, digits only |
| `phone_hashed` | string | MD5 hex digest of the normalized phone |
| `link` | string | Link in plain text, not validated |
| `profile_id` | integer | AmazingHiring profile id |

Plus an optional field:

| field | type | description |
| --- | --- | --- |
| `options` | object | Arbitrary object echoed back in every response and callback for this request |

Passing zero or more than one search field is a `400` validation error.

Request body:

```json
{
  "email": "dev@amazinghiring.com",
  "options": {
    "internal_candidate_id": 42
  }
}
```

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -H "X-Signature: ${SIGNATURE}" -X POST -d'{"email": "dev@amazinghiring.com", "options": {"internal_candidate_id": 42}}' "https://search.amazinghiring.com/api/dataenrichment/v1"
```

Response example (`201 Created`):

```json
{
  "rid": "0f8fad5b-d9cb-469f-a165-70867728950e",
  "code": 1,
  "message": "In Progress",
  "ts": "2026-08-18T09:12:31.482913",
  "profile": null,
  "options": {
    "internal_candidate_id": 42
  }
}
```

`ts` is the creation time in **UTC**, without a timezone suffix.

Possible errors: `400` (`101`), `401` (`103` / `201` / `202`), `403` (auth or signature),
`429` (`203`).

> The license is validated **before** the body is validated, so an expired license on a
> malformed request returns `401`, not `400`.

### Get enrichment request by id

**GET** `/v1/{rid}`

Returns the current state of the request. A request is only visible to the token that
created it.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/dataenrichment/v1/${RID}"
```

Response example (`200 OK`, still processing):

```json
{
  "rid": "0f8fad5b-d9cb-469f-a165-70867728950e",
  "code": 1,
  "message": "In Progress",
  "ts": "2026-08-18T09:12:31.482913",
  "profile": null,
  "options": {}
}
```

Response example (`200 OK`, finished): see [Profile object](#profile-object).

Response example (`404 Not Found`):

```json
{
  "code": 102,
  "message": "No request found for '0f8fad5b-d9cb-469f-a165-70867728950e'"
}
```

### Get quota

**GET** `/v1/quota`

Returns the license window and the number of searches left on it.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/dataenrichment/v1/quota"
```

Response example:

```json
{
  "searches_left": 143870,
  "from_date": "2021-01-01 00:00:00",
  "till_date": "2030-12-31 00:00:00"
}
```

If there is no license for your company the endpoint answers `401` with code `103`.

#### How searches are counted

* A search is charged **only when a profile is actually found** (`code = 0`).
  `Not Found`, `Failed` and `Timed Out` results are free.
* Repeated resolutions of the **same profile within the same UTC day** are charged once.
* If the profile is found but the license has no searches left, the request ends with
  status `203 Limits Exceeded` and no profile is returned.

## Result callback

If a callback url is registered for your account, the service `POST`s the finished
request to it as soon as it reaches a final status (`0`, `2`, `201`, `202`).

* Body: exactly the same JSON as `GET /v1/{rid}` returns.
* Header: `X-Signature`, computed over the body with your secret.
* Your endpoint must answer **exactly `200`** — any other status, including other `2xx`,
  is treated as a failure.
* On failure the callback is retried up to 4 times with a growing delay
  (2, 4, 8 and 16 minutes).
* Requests that end in `203 Limits Exceeded` do **not** trigger a callback — poll
  `GET /v1/{rid}` if you need to detect them.

Callback request example:

```http
POST /your/callback/path HTTP/1.1
Content-Type: application/json
X-Signature: 6c2a...f01d

{"rid": "0f8fad5b-d9cb-469f-a165-70867728950e", "code": 0, "message": "Success", "ts": "2026-08-18T09:12:31.482913", "profile": {...}, "options": {}}
```

## Timings and retries

| Stage | Behaviour |
| --- | --- |
| `profile_id` requests | Fetched from the backend 15 minutes after creation |
| Backend search call | Retried up to 10 times with a growing delay; after that the request becomes `201 Failed` |
| Backend result | Arrives asynchronously, usually within ~15 minutes |
| Stuck requests | After 1 hour without a result the request is re-issued (up to 4 backend attempts, at most one per hour) |
| Hard timeout | A request still `In Progress` after 24 hours becomes `202 Timed Out` |

## Profile object

`profile` is `null` until the request succeeds. Fields that the backend has no data for
are omitted.

> This profile format belongs to the Data Enrichment service and differs from the profile
> returned by the AmazingHiring API v6 — for example `skills` here is a tree of skill
> **groups**, courses carry a single `date`, and there are no `title`, `general_info`,
> `languages` or `all_skills_grouped` fields.

```json
{
  "rid": "0f8fad5b-d9cb-469f-a165-70867728950e",
  "code": 0,
  "message": "Success",
  "ts": "2026-08-18T09:12:31.482913",
  "options": {},
  "profile": {
    "id": 906901969936585,
    "name": "Brennan Edison",
    "nick": "amazing_username",
    "age": 30,
    "birthday": "1990-01-01",
    "avatars": [
      "https://avatars0.githubusercontent.com/u/amazing_username"
    ],
    "locations": [
      {
        "id": "london__greater-london__england__united-kingdom",
        "name": "London"
      }
    ],
    "contacts": [
      {
        "type": "email",
        "value": "dev@amazinghiring.com",
        "source": {
          "url": "https://github.com/amazing_username"
        }
      },
      {
        "type": "phone",
        "value": "+1 (000) 123 12 12",
        "source": {
          "url": null
        }
      }
    ],
    "links": [
      {
        "value": "https://github.com/amazing_username",
        "personal_site": false
      },
      {
        "value": "amazinghiring.com",
        "personal_site": true
      }
    ],
    "skills": [
      {
        "name": "Programming Languages",
        "sources": [],
        "additional_skills": [
          {
            "name": "Python",
            "sources": [
              {
                "url": "https://amazinghiring.com/"
              }
            ],
            "additional_skills": [
              {
                "name": "Django",
                "sources": [
                  {
                    "url": "https://amazinghiring.com/"
                  }
                ],
                "additional_skills": []
              }
            ]
          }
        ]
      }
    ],
    "comments": [
      {
        "id": 1,
        "body": "Comment text will be here",
        "user": {
          "id": 1,
          "name": "Amazing Recruiter"
        }
      }
    ],
    "educations": [
      {
        "name": "Amazing University",
        "short_name": "Amazing University",
        "specialization": "",
        "faculty": "",
        "degree": "BA (Hons) Media and Film Studies 2:2",
        "start": 1995,
        "end": 1996
      }
    ],
    "courses_or_certificates": [
      {
        "name": "Amazing programming course",
        "organization_name": "Amazing organization",
        "date": "2012-10-01T00:00:00"
      }
    ],
    "positions": [
      {
        "position": "Amazing python programmer",
        "description": "Amazing position description",
        "start": "2008-09",
        "end": "2022-09",
        "company": {
          "id": "AmazingHiring Global",
          "name": "AmazingHiring Global"
        }
      }
    ],
    "resumes": [
      {
        "id": "2",
        "type": "attached-file",
        "url": "ah://attached-file/Test_one.docx",
        "user": {
          "id": 1,
          "name": "Amazing Recruiter"
        }
      }
    ]
  }
}
```

### Field notes

| Field | Notes |
| --- | --- |
| `id` | AmazingHiring profile id |
| `birthday` | `YYYY-MM-DD`, only when the backend has an exact date |
| `contacts[].type` | `email`, `phone`, `skype`, … |
| `contacts[].source.url` | `null` when the contact comes from private (non-public) data |
| `links[].personal_site` | `true` if the link is the person's own website |
| `skills[]` | Skill groups; the concrete skills are in `additional_skills`, recursively |
| `courses_or_certificates[].date` | Issue/completion datetime in ISO format |
| `positions[].start` / `end` | `YYYY-MM`, `null` if unknown |
| `resumes[].url` | Can be an internal `ah://` url |
| `*.user` | AmazingHiring user; `id = -1` for system users |

## Monitoring

Two unauthenticated endpoints are exposed by both the public and the internal app:

* **GET** `/health` — liveness. Always `200` with the running version; deliberately has no
  dependency checks.
* **GET** `/status` — readiness. `200` when all dependencies are reachable, `500` otherwise.

```json
{
  "app_version": "1.4.2",
  "checks": {
    "db": "ok"
  }
}
```
