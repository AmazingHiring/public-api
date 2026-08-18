# AmazingHiring API v6

AmazingHiring API is a set of endpoints for integration of AmazingHiring with your product.

Current version: 6.

Base API url: **<https://search.amazinghiring.com/api/v6/>**

Interactive Swagger UI: **<https://search.amazinghiring.com/api/v6/docs/>**, machine
readable OpenAPI 3.0 schema: **<https://search.amazinghiring.com/api/v6/schema/>**. Both
are generated from the running service, so they always describe the current behaviour.

## Authorization

All API endpoints require an access token.

Remember to keep your token secret, treat it just like password! It acts on behalf of your company and has access to private data when interacting with the API.

### Access token generation

The token can be created through the web UI on user settings page,
which is **available after approval** from [AmazingHiring sales manager](mailto:sales@amazinghiring.com).

### Use the access token

The token key should be included in the `Authorization` header.
The key should be prefixed by the string literal "Token" \(or "Bearer" if your appliccatoin authorized via [oAuth2.0](https://amazinghiring.github.io/oauth2-docs/)\), with whitespace separating the two strings. For example:

```http
Authorization: Token a0b1c2d3e4f5
```

## Pagination

There are two pagination styles in the API:

* **page-based** with a `Link` header — the default, used by `/folders/` and profile endpoints;
* **limit/offset** with a wrapped response body — used by `/candidates/`
  \(see [Get all candidates](#get-all-candidates)\).

### How do I know if there are more pages?

If there are more pages, the response will be provided with `Link` header. Example:

```http
Link: <https://search.amazinghiring.com/api/v6/folders/?page=2>; rel="next", <https://search.amazinghiring.com/api/v6/folders/?page=7>; rel="last"
```

The header may also contain a `rel="prev"` link.

To get `N` page you have to pass page number via GET-parameter. Example:

```bash
https://search.amazinghiring.com/api/v6/folders/?page=2
```

Page size is 50 by default and can be changed with the `per_page` GET-parameter \(max 1000\):

```bash
https://search.amazinghiring.com/api/v6/folders/?page=2&per_page=200
```

## Profiles

* [**GET** `/profiles/{profile_id}/` Returns profile by id](#get-profile-by-id)

### GET Profile by id

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/profiles/${PROFILE_ID}/"
```

Response example:

```json
{
  "id": 906901969936585,
  "name": "Brennan Edison",
  "avatars": [
    "https://avatars0.githubusercontent.com/u/amazing_username",
  ],
  "age": 30,
  "birthday": "1990-01-01",
  "general_info": "Designing server-side business logic.",
  "title": "Backend Engineer",
  "comments": [
    {
      "id": 1,
      "body": "Comment text will be here",
      "created_at": "2020-10-01T16:30:36.552087+03:00",
      "modified_at": "2020-10-01T16:30:44.403443+03:00",
      "user": {
        "id": 1,
        "name": "Amazing Recruiter"
      },
    }
  ],
  "resumes": [
    {
      "id": 2,
      "type": "attached-file",
      "url": "ah://attached-file/Test_one.docx",
      "user": {
        "id": 1,
        "name": "Amazing Recruiter"
      }
    }
  ],
  "links": [
    {
      "value": "https://github.com/amazing_username",
      "personal_site": false,
    },
    {
      "value": "amazinghiring.com",
      "personal_site": true,
    },
  ],
  "contacts": [
    {
      "type": "phone",
      "value": "+1 (000) 123 12 12",

    },
    {
      "type": "email",
      "value": "dev@amazinghiring.com",
    }
  ],
  "locations": [
    {
        "id": "london__greater-london__england__united-kingdom",
        "name": "London"
    }
  ],
  "educations": [
    {
      "degree": "BA (Hons) Media and Film Studies 2:2",
      "end": 1996,
      "faculty": "",
      "name": "Amazing University",
      "specialization": "",
      "start": 1995
    },
  ],
  "languages": {
        "English": "Limited working proficiency",
        "Dutch": "Native or bilingual proficiency"
  },
  "positions": [
    {
      "company": {
          "id": "AmazingHiring Global",
          "name": "AmazingHiring Global",
          "site": null
      },
      "description": "Amazing position description",
      "position": "Amazing python programmer",
      "skills": null,
      "start": "2008-09",
      "end": "2022-09"
    }
  ],
  "courses_or_certificates": [
    {
      "name": "Amazing programming course",
      "organization_name": "Amazing organization",
      "start": "2012-01",
      "end": "2012-10"
    }
  ],
  "skills": [
    {
      "name": "python",
      "sources": [
        {
          "url": "https://amazinghiring.com/"
        }
      ],
      "additional_skills": [
        {
          "name": "django",
          "sources": [
            {
              "url": "https://amazinghiring.com/",
            }
          ],
          "additional_skills": []
        }
      ]
    }
  ],
  "all_skills_grouped": [
        {
            "id": "programming-languages",
            "name": "Programming Languages",
            "skills": [
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
        },
        {
            "id": "databases",
            "name": "Databases",
            "skills": [
                {
                    "name": "SQL",
                    "sources": [
                        {
                            "url": "https://amazinghiring.com/"
                        }
                    ],
                    "additional_skills": [
                        {
                            "name": "PostgreSQL",
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
    ]
}
```

Notes:

* `skills` contains only the "Programming Languages" group; use `all_skills_grouped` to get
  every skill group.
* `birthday` is an empty string if the exact date is unknown.
* `positions[].start` / `positions[].end` are `YYYY-MM` or `null`; `positions[].skills` is
  passed through from the source data as is and may be `null`.

## Folders

* [Folder object](#folder-object)
* [**GET** `/folders/` Returns folders list](#get-folders)
* [**GET** `/folders/{folder_id}/` Returns folder object by id](#get-folder-by-id)
* [**POST** `/folders/` Creates folder](#create-folder)
* [**PATCH** `/folders/{folder_id}/` Updates folder](#update-folder)
* [**DELETE** `/folders/{folder_id}/` Deletes folder](#delete-folder)

### Folder object

This is the format of a **single** folder \(`GET`/`POST`/`PATCH` on a folder\).
The folder **list** returns a slightly different set of fields, see
[Get folders](#get-folders).

Example of folder object:

```json
{
    "id": 1,
    "name": "Amazing folder",
    "description": "",
    "target_company": null,
    "status": "active",
    "access_type": "public",
    "assignees": [
        {
            "email": "user_email@amazinghiring.com",
            "first_name": "Amazing",
            "id": 1,
            "last_name": "Recruiter",
            "type": "employee"
        }
    ],
    "candidate_statuses": [
        {
            "candidates": [
                100001,
                100002
            ],
            "candidates_count": 2,
            "color": 248,
            "id": 100001,
            "name": "Contacted",
            "order": 100,
            "parent_id": null,
            "substatuses": [],
            "type": "default"
        },
        {
            "candidates": [],
            "candidates_count": 0,
            "color": 135,
            "id": 100002,
            "name": "Interested",
            "order": 200,
            "parent_id": null,
            "substatuses": [],
            "type": "default"
        },
        {
            "candidates": [],
            "candidates_count": 0,
            "color": 203,
            "id": 100003,
            "name": "Sourced",
            "order": 300,
            "parent_id": null,
            "substatuses": [],
            "type": "default"
        },
        {
            "candidates": [
                100006
            ],
            "candidates_count": 1,
            "color": 0,
            "id": 100009,
            "name": "Not Interested",
            "order": 400,
            "parent_id": null,
            "substatuses": [],
            "type": "default"
        },
        {
            "candidates": [],
            "candidates_count": 0,
            "color": 0,
            "id": 100010,
            "name": "status#1",
            "order": 600,
            "parent_id": null,
            "substatuses": [],
            "type": "custom"
        }
    ],
    "candidates_count": 3,
    "created_at": "2022-12-26T00:00:00.000832+03:00",
    "modified_at": "2023-01-25T00:00:00.441524+03:00",
    "creator": {
        "email": "user_email@amazinghiring.com",
        "first_name": "Amazing",
        "id": 28395,
        "last_name": "Recruiter",
        "type": "employee"
    },
    "granted_users": [],
    "mailing_lists": []
}
```

Notes:

* `candidate_statuses` is read-only. To set statuses use the write-only
  `candidate_status_ids` field \(see [Create folder](#create-folder)\).
* Only root statuses are returned in `candidate_statuses`; their children are in
  `substatuses`.
* `mailing_lists` contains the mailing lists of the folder; it is read-only and omitted
  here for brevity.
* `accessible` and `ats_type` are returned by the folder **list** only.

### Get folders

**GET** `/folders/`

Returns array of folder objects. Paginated with the `Link` header
\(see [Pagination](#pagination)\).

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/folders/"
```

Response example:

```json
[
 {
        "access_type": "public",
        "accessible": true,
        "assignees": [
            {
                "email": "user_email@amazinghiring.com",
                "first_name": "Amazing",
                "last_name": "Recruiter",
                "id": 1,
                "type": "employee"
            }
        ],
        "ats_type": null,
        "candidates_count": 619,
        "created_at": "2022-11-23T12:12:24.455126+03:00",
        "creator": {
            "email": "user_email@amazinghiring.com",
            "first_name": "Amazing",
            "last_name": "Recruiter",
            "id": 1,
            "type": "employee"
        },
        "description": "Folder description",
        "granted_users": [],
        "id": 1,
        "modified_at": "2023-05-23T15:54:31.787689+03:00",
        "name": "Folder name",
        "status": "active",
        "target_company": null
    },
]
```

Supported filters: `status`, `assignees`, `creator`, `created_at`. Ordering is by
`-modified_at` by default and can be changed with the `ordering` GET-parameter. Pass
`?only_accessible=true` to get only the folders available to you.

### Get folder by id

**GET** `/folders/{folder_id}/`

Returns folder objects by id.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/folders/1/"
```

[Response format](#folder-object)

### Create folder

**POST** `/folders/`

Creates new folder. At least one assignee and at least one candidate status are required.

Request body:

```json
{
    "access_type": "public",
    "assignees": [
        1
    ],
    "candidate_status_ids": [
        10001,
        10002,
        10003
    ],
    "description": "Folder description",
    "granted_users": [],
    "name": "Amazing folder",
    "status": "active"
}
```

Returns created folder object with HTTP code 201.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X POST -d'{"access_type": "public", "assignees": [1], "candidate_status_ids": [10001, 10002, 10003], "name": "Amazing folder", "description": "Folder description", "granted_users": [], "status": "active"}' "https://search.amazinghiring.com/api/v6/folders/"
```

[Response format](#folder-object)

### Update folder

**PATCH** `/folders/{folder_id}/`

Updates existing folder. Only the fields you send are changed.

Request body:

```json
{
    "access_type": "public",
    "assignees": [
        1
    ],
    "candidate_status_ids": [
        10001,
        10002,
        10003
    ],
    "description": "Folder description",
    "granted_users": [],
    "name": "Amazing folder",
    "status": "active"
}
```

Returns updated folder object.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X PATCH -d'{"name": "Amazing folder", "description": "Folder description"}' "https://search.amazinghiring.com/api/v6/folders/${FOLDER_ID}/"
```

[Response format](#folder-object)

### Delete folder

**DELETE** `/folders/{folder_id}/`

Deletes existing folder.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X DELETE "https://search.amazinghiring.com/api/v6/folders/${FOLDER_ID}/"
```

Response will be empty with HTTP code 204.

## Candidates

Candidate object links profile to folder with status. There will be 2 candidate objects if you will add 1 profile to 2 different folders.

* [Candidate object](#candidate-object)
* [**GET** `/candidates/` Returns candidates list](#get-all-candidates)
* [**GET** `/candidates/{candidate_id}/ Returns candidate object by id`](#get-candidate-by-id)
* [**POST** `/candidates/ Creates candidate`](#create-candidate)
* [**PATCH** `/candidates/{candidate_id}/ Updates candidate`](#update-candidate)
* [**DELETE** `/candidates/{candidate_id}/ Deletes candidate`](#delete-candidate)

### Candidate object

On read, `folder` is expanded to a [folder object](#get-folders) and `status` to the status
path. On write, both are plain ids.

```json
{
    "id": 1,
    "profile": 1111000001111,
    "creator": 1,
    "created_at": "2016-04-15T10:39:41.079931Z",
    "updated_at": "2016-04-15T10:39:41.079931Z",
    "folder": {
        "access_type": "public",
        "accessible": true,
        "assignees": [],
        "ats_type": null,
        "candidates_count": 1,
        "created_at": "2022-08-17T16:18:56.308039+03:00",
        "creator": {
            "email": "user_email@amazinghiring.com",
            "first_name": "Amazing",
            "last_name": "Recruiter",
            "id": 1,
            "type": "employee"
        },
        "description": "Folder description",
        "granted_users": [],
        "id": 1,
        "modified_at": "2023-06-01T17:06:42.147729+03:00",
        "name": "Folder name",
        "status": "active",
        "target_company": null
    },
    "status": {
        "id": 44376,
        "name": "Contacted",
        "color": 248,
        "substatus": null
    },
    "comments": [],
    "recipients_count": 0,
    "search_query": null
}
```

| Field | Type | Notes |
| --- | --- | --- |
| `id` | integer | Candidate id, read-only |
| `profile` | integer | AmazingHiring profile id. Cannot be changed after creation |
| `folder` | integer on write, object on read | Required on create |
| `status` | integer on write, object on read | Status id; `substatus` is nested inside `status` on read |
| `creator` | integer | User id, read-only |
| `created_at` / `updated_at` | datetime | Read-only |
| `comments` | array | Read-only, empty if the folder is not accessible to you |
| `recipients_count` | integer | Read-only, number of mailing recipients |
| `search_query` | integer | Search query id or `null` |

### Get all candidates

**GET** `/candidates/`

Returns list of all candidates in your company.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/candidates/"
```

This endpoint uses limit/offset pagination \(**not** the `Link` header\):

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/candidates/?limit=100&offset=1200"
```

You can request candidates from specific folder

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/folders/${FOLDER_ID}/candidates/"
```

By default every candidate in the list is returned with the full profile inlined into the
`profile` field. Pass `?short=true` to keep `profile` as a plain profile id, which is much
faster.

Response example:

```json
{
  "count": 20924,
  "next": "https://search.amazinghiring.com/api/v6/candidates/?limit=1&offset=1",
  "previous": null,
  "results": [
    {
      "id": 1,
      "profile": "Profile info will be here in json format",
      "creator": 1,
      "created_at": "2023-06-01T17:06:42.137149+03:00",
      "updated_at": "2023-06-01T17:06:42.137166+03:00",
      "folder": {
          "access_type": "public",
          "accessible": true,
          "assignees": [
              {
                  "email": "user_email@amazinghiring.com",
                  "first_name": "Amazing",
                  "last_name": "Recruiter",
                  "id": 1,
                  "type": "employee"
              }
          ],
          "ats_type": null,
          "candidates_count": 1,
          "created_at": "2022-08-17T16:18:56.308039+03:00",
          "creator": {
              "email": "user_email@amazinghiring.com",
              "first_name": "Amazing",
              "last_name": "Recruiter",
              "id": 1,
              "type": "employee"
          },
          "description": "Folder description",
          "granted_users": [],
          "id": 1,
          "modified_at": "2023-06-01T17:06:42.147729+03:00",
          "name": "Folder name",
          "status": "active",
          "target_company": null
      },
      "status": {
          "id": 44376,
          "name": "Contacted",
          "color": 248,
          "substatus": null
      },
      "comments": [],
      "recipients_count": 0,
      "search_query": null
    }
  ]
}
```

### Get candidate by id

**GET** `/candidates/{candidate_id}/`

Returns candidate object by id. Unlike the list, `profile` here is the profile id.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/candidates/${CANDIDATE_ID}/"
```

[Response format](#candidate-object)

### Create candidate

**POST** `/candidates/`

Adds profile to folder. Returns created candidate object with HTTP code 200.

Request body:

```json
{
  "profile": 1111000001111,
  "folder": 1
}
```

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X POST -d'{"profile": '${PROFILE_ID}', "folder": '${FOLDER_ID}'}' "https://search.amazinghiring.com/api/v6/candidates/"
```

[Response format](#candidate-object)

#### Bulk create

If the request body is an **array**, several candidates are created at once. Invalid items
\(for example, a profile already present in that folder\) are skipped, the valid ones are
still saved, and the response is a summary instead of a candidate object.

Request body:

```json
[
  {
    "profile": 1111000001111,
    "folder": 1
  },
  {
    "profile": 1111000002222,
    "folder": 1
  }
]
```

Response example:

```json
{
    "successCount": 2,
    "invalidCount": 0
}
```

### Update candidate

**PATCH** `/candidates/{candidate_id}/`

Updates candidate object. The most common use is moving a candidate to another status —
pass the status id in the `status` field. `profile` cannot be changed.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X PATCH -d'{"status": 44377}' "https://search.amazinghiring.com/api/v6/candidates/${CANDIDATE_ID}/"
```

[Response format](#candidate-object)

### Delete candidate

**DELETE** `/candidates/{candidate_id}/`

Deletes candidate object

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -X DELETE "https://search.amazinghiring.com/api/v6/candidates/${CANDIDATE_ID}/"
```

Response will be empty with HTTP code 204.
