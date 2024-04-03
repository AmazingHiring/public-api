# AmazingHiring API v6

AmazingHiring API is a set of endpoints for integration of AmazingHiring with your product.

Current version: 6.

Base API url: **<https://search.amazinghiring.com/api/v6/>**

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

### How do I know if there are more pages?

If there are more pages, the response will be provided with `Link` header. Example:

```http
Link: <https://search.amazinghiring.com/api/v6/candidates/?page=2; rel="next">, <https://search.amazinghiring.com/api/v6/candidates/?page=7; rel="last">
```

To get `N` page you have to pass page number via GET-parameter. Example:

```bash
https://search.amazinghiring.com/api/v6/candidates/?page=2
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
      "created_at": "2015-09-10T05:59:57.660Z",
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
      "short_name": "Amazing University",
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
      "end": "2008-09",
      "position": "Amazing python programmer",
      "start": "2022-09"
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
          "rating": 4.1,
          "sources": [
            {
              "url": "https://amazinghiring.com/",
            }
          ]
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

## Folders

* [Folder object](#folder-object)
* [**GET** `/folders/` Returns folders list](#get-folders)
* [**GET** `/folders/{folder_id}/` Returns folder object by id](#get-folder-by-id)
* [**POST** `/folders/` Creates folder](#create-folder)
* [**PATCH** `/folders/{folder_id}/` Updates folder](#update-folder)
* [**DELETE** `/folders/{folder_id}/` Deletes folder](#delete-folder)

### Folder object

Example of folder object:

```json
{
    "name": "Amazing folder",
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
    "description": "",
    "granted_users": [],
    "id": 1
}
```

### Get folders

**GET** `/folders/`

Returns array of folder objects.

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
        "status": "active"
    },
]
```

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

Creates new folder.

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

Returns created folder object.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X POST -d'{"access_type": "public", "assignees": [1], "candidate_status_ids": [10001, 10002, 10003], "name": "Amazing folder", "description": "Folder description", "granted_users": [], "status": "active"}' "https://search.amazinghiring.com/api/v6/folders/"
```

[Response format](#folder-object)

### Update folder

**PATCH** `/folders/{folder_id}/`

Updates existing folder.

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

Returns created folder object.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X PATCH -d'{"access_type": "public", "assignees": [1], "candidate_status_ids": [10001, 10002, 10003], "name": "Amazing folder", "description": "Folder description", "granted_users": [], "status": "active"}' "https://search.amazinghiring.com/api/v6/folders/"
```

[Response format](#folder-object)

### Delete folder

**DELETE** `/folders/{folder_id}/`

Deletes existing folder.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X DELETE "https://search.amazinghiring.com/api/v6/folders/${FOLDER_ID}"
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

```json
{
    "created_at": "2016-04-15T10:39:41.079931Z",
    "creator": 1,
    "folder_id": 1,
    "id": 1,
    "profile": 1,
    "status_id": 2
}
```

### Get all candidates

**GET** `/candidates/`

Returns list of all candidates in your company.

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/candidates/"
```

Pagination is supported via limit and offset query parameters.

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/candidates/?limit=100&offset=1200"
```

You can request candidates from specific folder

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/folders/${FOLDER_ID}/candidates/"
```

Response example:

```json
[
  {
    "count": 20924,
    "next": "https://search.amazinghiring.com/api/v6/candidates/?limit=1&offset=1",
    "previous": null,
    "results": [
      {
        "comments": [],
        "creator": 1,
        "created_at": "2023-06-01T17:06:42.137149+03:00",
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
            "creator": {
                "email": "user_email@amazinghiring.com",
                "first_name": "Amazing",
                "last_name": "Recruiter",
                "id": 1,
                "type": "employee"
            },
            "id": 1,
            "granted_users": [],
            "created_at": "2022-08-17T16:18:56.308039+03:00",
            "modified_at": "2023-06-01T17:06:42.147729+03:00",
            "name": "Folder name",
            "description": "Folder description",
            "status": "active"
        },
        "id": 1,
        "profile": "Profile info will be here in json format",
        "recipients_count": 0,
        "search_query": null,
        "status": {
            "color": 248,
            "id": 44376,
            "name": "Contacted",
            "substatus": null
        },
        "updated_at": "2023-06-01T17:06:42.137166+03:00"
      }
    ]
  }
]
```

### Get candidate by id

**GET** `/candidates/{candidate_id}/`

Returns candidate object by id

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Accept: application/json" "https://search.amazinghiring.com/api/v6/candidates/${CANDIDATE_ID}/"
```

Response example:

```json
{
    "comments": [],
    "created_at": "2023-06-01T17:06:42.137149+03:00",
    "creator": 1,
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
        "status": "active"
    },
    "id": 1,
    "profile": 1111000001111,
    "recipients_count": 0,
    "search_query": null,
    "status": {
        "color": 248,
        "id": 44376,
        "name": "Contacted",
        "substatus": null
    },
    "updated_at": "2023-06-01T17:06:42.137166+03:00"
}
```

### Create candidate

**POST** `/candidates/`

Adds profile to folder. Returns created candidate object

Request body:

```json
{
  [
    {
      "profile": 1, 
      "folder": 1
    }
  ]
}
```

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X POST -d'{"profile": ${PROFILE_ID}, "folder": ${FOLDER_ID}}' "https://search.amazinghiring.com/api/v6/candidates/"
```

Response example:

```json
{
    "comments": [],
    "created_at": "2023-06-01T17:06:42.137149+03:00",
    "creator": 1,
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
        "status": "active"
    },
    "id": 1,
    "profile": 1111000001111,
    "recipients_count": 0,
    "search_query": null,
    "status": {
        "color": 248,
        "id": 44376,
        "name": "Contacted",
        "substatus": null
    },
    "updated_at": "2023-06-01T17:06:42.137166+03:00"
}
```

### Update candidate

**PATCH** `/candidates/{candidate_id}/`

Updates candidate object

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -H "Content-Type: application/json" -X PATCH -d'{"status_id": 8}' "https://search.amazinghiring.com/api/v6/candidates/${CANDIDATE_ID}/"
```

Response example:

```json
{
    "comments": [],
    "created_at": "2023-06-01T17:06:42.137149+03:00",
    "creator": 1,
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
        "status": "active"
    },
    "id": 1,
    "profile": 1111000001111,
    "recipients_count": 0,
    "search_query": null,
    "status": {
        "color": 248,
        "id": 44376,
        "name": "Contacted",
        "substatus": null
    },
    "updated_at": "2023-06-01T17:06:42.137166+03:00"
}
```

### Delete candidate

**DELETE** `/candidates/{candidate_id}/`

Deletes candidate object

Request example:

```bash
curl -H "Authorization: Token ${TOKEN}" -X DELETE "https://search.amazinghiring.com/api/v6/candidates/${CANDIDATE_ID}/"
```

Response will be empty with HTTP code 200.
