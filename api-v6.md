# AmazingHiring API v6

AmazingHiring API is a set of endpoints for integration of AmazingHiring with your product.

* [Profiles](#profiles)
* [Profile search](#profile-search)
* [Folders](#folders)
* [Candidates](#candidates)

Current version: 6.

Base API url: **<https://search.amazinghiring.com/api/v6/>**

Interactive Swagger UI: **<https://search.amazinghiring.com/api/v6/docs/>**, machine
readable OpenAPI 3.0 schema: **<https://search.amazinghiring.com/api/v6/schema/>**. Both
are generated from the running service, so they always describe the current behaviour.

## Authorization

All API endpoints require an access token.

Remember to keep your token secret, treat it just like password! It acts on behalf of your company and has access to private data when interacting with the API.

### Access token generation

There is one token per company. A company administrator creates it in the web UI on the
**Company → API management** page, which is **available after approval** from
[AmazingHiring sales manager](mailto:sales@amazinghiring.com). Regenerating the token
disables every integration of the company.

[Profile search](#search-access) is enabled for a company separately, by the same manager;
its current state is shown on the same **API management** page.

### Use the access token

The token key should be included in the `Authorization` header.
The key should be prefixed by the string literal "Token" \(or "Bearer" if your appliccatoin authorized via [oAuth2.0](https://amazinghiring.github.io/oauth2-docs/)\), with whitespace separating the two strings. For example:

```http
Authorization: Token a0b1c2d3e4f5
```

## Pagination

There are two pagination styles in the API:

* **page-based** with a `Link` header — the default, used by `/folders/`;
* **limit/offset** with a wrapped response body — used by `/candidates/`
  \(see [Get all candidates](#get-all-candidates)\).

`POST /profiles/search/` takes `page` and `per_page` in the request body, see
[Pagination and the search limit](#pagination-and-the-search-limit).

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
* [**POST** `/profiles/search/` Searches profiles by a query](#profile-search)
* [**GET** `/suggestions/` Completes a name to a search term](#suggestions--get-suggestions)

### GET Profile by id

Returns the full profile, including contacts. Requesting a profile whose contacts your
company has not opened yet spends 1 credit of the contacts quota; repeated requests for the
same profile are free. A daily limit of profiles opened through the API also applies.
When the quota or the limit is exhausted the endpoint answers `402`. See
[Obtaining contacts](#obtaining-contacts).

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

## Profile search

* [**POST** `/profiles/search/` Searches profiles by a query](#request-structure)
* [**GET** `/suggestions/` Completes a name to a search term](#suggestions--get-suggestions)

`POST /profiles/search/` finds candidates in the AmazingHiring database by a JSON query:
skills, positions, companies, locations and the same filters the search page offers. Results
carry no contacts; contacts are obtained with a separate
[`GET /profiles/{id}/`](#get-profile-by-id) request. Slugs of the entities for a query come
from [`GET /suggestions/`](#suggestions--get-suggestions).

### Search access

Search uses the same token as the rest of the API, but it is enabled for a company
separately, by your [AmazingHiring sales manager](mailto:sales@amazinghiring.com). Its
current state is shown on the **Company → API management** page. Without it both
`POST /profiles/search/` and `GET /suggestions/` answer `402` with code `1012`.

* Server-to-server only: CORS is not configured, calling from a browser is not possible.
* Requests are synchronous; a search may take several seconds.
* Every closed list of values (enum) is in the [OpenAPI schema](https://search.amazinghiring.com/api/v6/schema/),
  see [Where to get the values](#where-to-get-the-values).
* [Swagger UI](https://search.amazinghiring.com/api/v6/docs/) lets you call the endpoints
  right from the page: press **Authorize**, choose `TokenAuth`, paste `Token a0b1c2d3e4f5` —
  then **Try it out** on any operation.

### Quick start

Python developers in and around Berlin with a known email.

First the slug of the location — suggestions by the beginning of a name:

```bash
curl "https://search.amazinghiring.com/api/v6/suggestions/?type=location&q=berlin" \
  -H "Authorization: Token a0b1c2d3e4f5"
```

```json
{"results": [{"value": "id-berlin__berlin__germany", "name": "Berlin, Germany"}, ...]}
```

`value` of the first match goes into the search request as is:

```bash
curl -X POST https://search.amazinghiring.com/api/v6/profiles/search/ \
  -H "Authorization: Token a0b1c2d3e4f5" \
  -H "Content-Type: application/json" \
  -d '{
    "skills": {"all": ["python"]},
    "locations": {"any": [{"value": "id-berlin__berlin__germany", "radius_km": 40}]},
    "filters": {"contacts": {"any": ["email"]}},
    "per_page": 20
  }'
```

Response:

```json
{
  "count": 1342,
  "page": 1,
  "per_page": 20,
  "results": [
    {
      "id": 6904,
      "name": "Guido van Rossum",
      "title": "Engineer at Dropbox",
      "locations": [{"id": "berlin__berlin__germany", "name": "Berlin, Germany"}],
      "positions": [...],
      "contacts_opened": false,
      ...
    }
  ]
}
```

Next — `GET /api/v6/profiles/6904/` to obtain the contacts (see [Obtaining contacts](#obtaining-contacts)).


### Request structure

```json
{
  "skills":         {"all": [...], "any": [...], "none": [...], "preferred": [...]},
  "positions":      {"any": [...], "none": [...]},
  "last_positions": {"any": [...], "none": [...]},
  "companies":      {"any": [...], "none": [...]},
  "last_companies": {"any": [...], "none": [...]},
  "locations":      {"any": [...], "none": [...]},
  "educations":     {"any": [...], "none": [...]},
  "names":          {"any": [...], "none": [...]},
  "text":           {"any": [...], "none": [...]},

  "filters": { ... },

  "page": 1,
  "per_page": 50
}
```

Three parts:

1. **Terms** (`skills`, `positions`, … `text`) — *what* to search for. At least one positive term (`all`, `any` or `preferred`) in any group is required. A request with filters only returns `400`.
2. **Filters** (`filters`) — *how* to narrow down. All optional.
3. **Pagination** (`page`, `per_page`).

The request is strict: an unknown key at any level is a `400` error with the text `Unknown field.`. A typo never silently turns into an empty result.


### Terms

#### What a term is

A term is a string or an object `{"value": "...", ...}`. The two forms are equivalent; the object is needed when there is a qualifier (`min_years`, `radius_km`).

```json
"python"
{"value": "python"}
{"value": "id-python", "min_years": 3}
```

The value is one of two kinds:

| Kind | Example | How it is matched |
|---|---|---|
| **Slug of a known entity** — starts with `id-` | `id-python`, `id-google`, `id-berlin__berlin__germany` | Exact match of a database entity: every synonym and spelling. More reliable. |
| **Free text** | `python`, `backend engineer`, `machine learning` | Full-text search. Several words are matched as a phrase. |

Where to get slugs — see [Where to get the values](#where-to-get-the-values).

Constraints on the value:

- 1 to 100 characters;
- no comma;
- does not start with `-`.

Each list (`all`, `any`, …) holds at most 50 terms.

#### Operator logic

| Key | Meaning | Available in |
|---|---|---|
| `all` | Every term must match (AND) | `skills` only |
| `any` | At least one term must match (OR) | every group |
| `none` | No term may match (NOT) | every group |
| `preferred` | Does not filter, only ranks matches higher | `skills` only |

Different groups are joined with AND: `skills.all = [python]` + `locations.any = [id-germany]` — Python **and** Germany.

#### Term groups

| Group | What is searched | Notes |
|---|---|---|
| `skills` | Skills | `all` supports `min_years` |
| `positions` | Positions over the whole career | |
| `last_positions` | Current position | |
| `companies` | Companies over the whole career | |
| `last_companies` | Current company | |
| `locations` | Place of residence: country, region or city | `any` supports `radius_km` |
| `educations` | Schools and universities | |
| `names` | First and last name | |
| `text` | Free text over the whole profile | |

#### Term qualifiers

**`min_years`** — least years of experience with the skill. Only in `skills.all`. Allowed values: `2`, `3`, `4`, `5`.

```json
"skills": {"all": [{"value": "id-go", "min_years": 3}]}
```

**`radius_km`** — also search within a radius around the city. Only in `locations.any`, cities only (ignored for a country or region). Allowed values: `20`, `40`, `80`, `160`.

```json
"locations": {"any": [{"value": "id-munich__bavaria__germany", "radius_km": 80}]}
```

Any other `min_years` / `radius_km` value is rejected with `400`.


### Filters

The `filters` object. Every field is optional.

#### Full list

##### Ranges

Format: `{"min": N, "max": N, "include_unknown": bool}`. Either bound is optional, both inclusive. `min > max` is an error. `include_unknown: true` adds profiles whose value is unknown to the results.

| Field | Unit |
|---|---|
| `age` | years |
| `experience_years` | total experience, years |
| `months_on_last_position` | months in the current position |
| `graduation_year` | calendar year of graduation |

```json
"experience_years": {"min": 5},
"age": {"min": 25, "max": 45, "include_unknown": true}
```

##### Closed lists (enum) — `{"any": [...], "none": [...]}`

`any` — at least one of the values, `none` — none of them.

| Field | Values | Note |
|---|---|---|
| `contacts` | `email`, `phone`, `messengers`, `any`, `no` | `any` — has some contact, `no` — has no contacts at all |
| `seniority` | `junior`, `middle`, `senior`, `team_lead`, `head_of`, `c_level`, `any`, `no` | Level of the current position. `any` — the level is known, `no` — unknown |
| `education_levels` | `bachelor`, `master`, `specialist`, `doctorate`, `csdegree` | Highest degree. `csdegree` — a Computer Science degree of any level |
| `company_size` | `1-10`, `11-200`, `201-500`, `501-1000`, `1001-5000`, `5001-10'000`, `10'000+` | Plus the `scope` field, see below |
| `gender` | `female` | Depends on the GDPR settings of the company |
| `diversity` | `asian`, `black`, `hispanic`, `indian`, `middleEast`, `white` | Depends on the GDPR settings of the company |

`company_size` additionally takes `scope` — which companies to consider:

| `scope` | Meaning |
|---|---|
| `current` | current company (default) |
| `previous` | previous companies |
| `any` | any |

```json
"company_size": {"any": ["201-500", "501-1000"], "scope": "any"}
```

##### Open lists — `{"any": [...], "none": [...]}`

Values are identifiers from the AmazingHiring database (see [Where to get the values](#where-to-get-the-values)).

| Field | What | Example value |
|---|---|---|
| `industries` | Industries of the companies | `software engineering` |
| `companies` | Companies over the whole career, by slug | `id-google` |
| `last_companies` | Current company, by slug | `id-microsoft` |
| `educations` | Schools and universities, by slug | `id-stanford-university` |

How `filters.companies` differs from the `companies` term: the term is a field of the search bar (a slug or free text), the filter is the checkboxes of the filter panel in the UI and takes only identifiers from it. If you know the slug, use the term; the filter exists to reproduce a query built through the filter panel.

##### Profile sources — `sites`

`{"all": [...], "none": [...]}` — domains the profile was collected from. `all` — the profile exists on every listed site (AND), `none` — on none of them.

```json
"sites": {"all": ["github.com", "stackoverflow.com"], "none": ["linkedin.com"]}
```

##### Languages — `languages`

A list of objects. Every listed language is **required** (AND between languages); levels within a language are OR.

```json
"languages": [
  {"name": "English", "levels": ["full", "native"]},
  {"name": "German"}
]
```

| Field | Values |
|---|---|
| `name` | Name of the language in English: `English`, `German`, `Spanish`… |
| `levels` | `elem` (elementary), `limited` (limited working), `prof` (professional working), `full` (full professional), `native`, `unknown`. Optional — without `levels` any level matches |

##### Flags — `true` / `false`

| Field | `true` means |
|---|---|
| `remote` | Open to remote work |
| `freelancer` | Freelancer |
| `frequent_job_changer` | Changes jobs more often than usual |
| `exclude_multiple_locations` | Exclude profiles whose sources disagree on the location |
| `exclude_linkedin` | Exclude profiles found on LinkedIn only |

`false` — explicitly require the opposite. To not filter by a flag, simply do not pass it.

#### Where to get the values

##### Closed lists

The values are listed above and fixed in the OpenAPI schema `GET /api/v6/schema/` — the `components.schemas` section:

| Schema | Field |
|---|---|
| `ContactKindEnum` | `filters.contacts` |
| `SeniorityEnum` | `filters.seniority` |
| `EducationLevelEnum` | `filters.education_levels` |
| `CompanySizeEnum`, `CompanySizeScopeEnum` | `filters.company_size` |
| `GenderEnum` | `filters.gender` |
| `DiversityEnum` | `filters.diversity` |
| `LanguageLevelEnum` | `filters.languages[].levels` |

Allowed `min_years` and `radius_km` are in the `SkillTermField` / `LocationTermField` schemas (also `enum`). The schema is the source of truth: if a list grows, it changes there.

##### Slugs (`id-...`)

A slug is `id-` + the identifier of an entity in the AmazingHiring database. Ways to get one:

1. **Suggestions — `GET /api/v6/suggestions/`** (see [Suggestions](#suggestions--get-suggestions)). The primary way: pass the beginning of a name and the kind of entity, get the list of known entities with a ready-to-use `value` term.

2. **From search results.** The identifiers in the response are the same entities without the prefix:
   - `results[].locations[].id` → `"berlin__berlin__germany"` → term `id-berlin__berlin__germany`;
   - `results[].positions[].company.id` → `"yandex"` → term `id-yandex`.

3. **From the address bar of the search page.** Build a query in the AmazingHiring UI through its suggestions — the URL of the results page carries `q=` parameters with the same slugs (`location[0]:id-united-states`, `skillAll[0]:id-python`).

4. **Free text.** If the slug is unknown, pass the name as text: `"python"`, `"google"`, `"berlin"`. It works, but catches only the literal spelling.

The format of location slugs is `city__region__country`; for a country just `id-germany`, for a region `id-bavaria__germany`. Slugs of companies and schools are Latin letters joined with hyphens: `id-google`, `id-stanford-university`.

An unknown slug raises no error — it simply matches nothing, and the results are empty or incomplete. Check `count`.

##### Industries, sites, languages

- `industries` — lowercase English names as in the filter panel of the UI: `software engineering`, `fintech`. There is no public list (`/suggestions/` has no industries); take them from the URL of the search page (`f=industry[0]:...`).
- `sites` — domain names: `github.com`, `stackoverflow.com`, `linkedin.com`, `habr.com`.
- `languages[].name` — the English name of the language, capitalized.

#### Suggestions — `GET /suggestions/`

Autocomplete of the entities the search understands. Pass the kind and the beginning of a name — get the known entities, best matches first.

```bash
curl "https://search.amazinghiring.com/api/v6/suggestions/?type=location&q=berl" \
  -H "Authorization: Token a0b1c2d3e4f5"
```

```json
{
  "results": [
    {"value": "id-berlin__berlin__germany", "name": "Berlin, Germany"},
    {"value": "id-berlin__coos-county__new-hampshire__united-states", "name": "Berlin, New Hampshire, USA"}
  ]
}
```

| Parameter | Required | Values |
|---|---|---|
| `type` | yes | `skill`, `position`, `company`, `location`, `education` |
| `q` | yes | Beginning of a name, 1–100 characters, no `,` |

| `type` | What it completes | Where `value` goes |
|---|---|---|
| `skill` | Skills and technologies | `skills.all/any/none/preferred` |
| `position` | Job titles | `positions`, `last_positions` |
| `company` | Companies | `companies`, `last_companies`, `filters.companies`, `filters.last_companies` |
| `location` | Cities, regions, countries | `locations` |
| `education` | Schools and universities | `educations`, `filters.educations` |

- `value` — a ready-to-use term: put it into a search request as is, without processing. For locations, companies and schools it is an `id-...` slug; for skills and positions it is the canonical name, which the search matches to the entity rather than to the spelling.
- `name` — a human-readable name to show to a user.
- An empty `results` means nothing matched. The language of the suggestions follows the language of the token's user.
- Suggestions do not spend the daily search limit. Cache the results on your side: the dictionary changes rarely.


### Search response

```json
{
  "count": 1342,
  "page": 1,
  "per_page": 50,
  "results": [ { ...profile... } ]
}
```

| Field | Meaning |
|---|---|
| `count` | Total number of matching profiles |
| `page`, `per_page` | As in the request (or the defaults) |
| `results` | Profiles of the current page |

#### Profile fields in the results

The same structure as [`GET /api/v6/profiles/{id}/`](#get-profile-by-id), minus `contacts` and `comments`, plus `contacts_opened`.

| Field | Type | Description |
|---|---|---|
| `id` | int | Profile identifier — for `GET /profiles/{id}/` |
| `name` | string | Name |
| `title` | string \| null | Profile headline (usually "position at company") |
| `general_info` | string \| null | Short summary |
| `age` | int \| null | Age |
| `birthday` | string | `YYYY-MM-DD`, an empty string when the date is unknown |
| `avatars` | string[] \| null | Photo URLs |
| `languages` | object | `{"English": "native", ...}` |
| `locations` | object[] | `{id, name}` — the id works as a slug with the `id-` prefix |
| `positions` | object[] | `{position, description, company: {id, name, site}, start, end, skills}`; current ones first; dates are `YYYY-MM` |
| `educations` | object[] | `{name, faculty, specialization, degree, start, end}` |
| `courses_or_certificates` | object[] | `{name, organization_name, start, end}` |
| `skills` | object[] | Programming languages only: `{name, sources[], additional_skills[]}` |
| `all_skills_grouped` | object[] | All skills by group: `{id, name, skills[]}` |
| `links` | object[] | `{value, personal_site}` — links to the sources |
| `resumes` | object[] | Resumes uploaded by your company |
| `contacts_opened` | bool | Contacts are already opened by your company — `GET /profiles/{id}/` is free |

**Links are masked.** By default `links[].value` is not the direct URL of the source but a proxy link like `https://search.amazinghiring.com/api/profiles/{id}/links/{hash}/`. It redirects to the original when opened. This is a company setting; to receive direct URLs, contact your AmazingHiring manager.

Profiles that requested deletion of their data (opt-out) are not returned — so a page may hold fewer than `per_page` items while the next pages are not empty.


### Obtaining contacts

Contacts are the paid part. Search does not return them.

```bash
curl https://search.amazinghiring.com/api/v6/profiles/6904/ \
  -H "Authorization: Token a0b1c2d3e4f5"
```

The response is the full profile with the `contacts` field:

```json
"contacts": [
  {"type": "email", "value": "guido@example.com"},
  {"type": "phone", "value": "+1..."},
  {"type": "skype", "value": "..."}
]
```

Billing rules:

- If the search results show `contacts_opened: true`, the contacts are already opened by your company and the request is free.
- If `false`, **1 credit of the company's contacts quota** is spent. Repeated requests for the same profile are free.
- Separately, a daily limit of profiles opened through the API applies (400 per company per day by default).
- Limit exhausted → `402` (see [Search errors](#search-errors)).

The `GET /profiles/{id}/` request *is* "open the contacts". Do not call it "just to look": the charge happens on GET.

Whether opening is worth it can be checked with the `contacts` filter in the search itself: `{"contacts": {"any": ["email"]}}` returns only profiles with a known email.


### Pagination and the search limit

| Parameter | Default | Range |
|---|---|---|
| `page` | 1 | ≥ 1 |
| `per_page` | 50 | 1–100 |

To walk the whole result set, increase `page` while `page * per_page < count`.

**Every received page spends the company's daily limit of search results** (15,000 profiles a day by default). The number of profiles actually returned is charged. When exhausted — `429`; the limit resets a day after the last request.

In practice:

- Use `per_page: 100` — the limit is spent on profiles, not on requests; large pages are simply faster.
- Repeating a **byte-for-byte identical** request (same terms, filters, `page`, `per_page`) does not spend the limit again — the history entry is overwritten. Any difference is a new request.
- Narrow the query with filters down to the `count` you need instead of exporting everything.

The limit is counted per the token's user. The company token is a separate service user, so manual searches by employees in the UI do not interfere with the integration and vice versa.

`GET /suggestions/` does not count towards the limit of results.


### Search errors

| Code | Cause | What to do |
|---|---|---|
| `400` | Invalid request: unknown field, invalid enum value, `min > max`, no positive term, `per_page > 100` | Fix the request. The body names the offending field |
| `401` | No `Authorization` header, wrong token, or the company has no API access | Check the token |
| `403` | No active license, or a GDPR restriction (see below) | Contact the company administrator |
| `402` | Contacts quota or the daily limit of profiles through the API is exhausted (`GET /profiles/{id}/`), or the Search API is not enabled for the company (`POST /profiles/search/`, `GET /suggestions/`, code `1012`) | Wait for the reset / raise the quota; for `1012` contact your AmazingHiring manager |
| `429` | The daily limit of search results is exhausted | Retry in a day |
| `502` | The search backend is unavailable or returned an error | Retry with exponential backoff (1 s, 2 s, 4 s…) |
| `504` | The search backend did not answer in time | Retry with a delay; simplify the query |

#### Validation error format (`400`)

Standard: the key is the path to the field, the value is a list of messages.

```json
{"skill": ["Unknown field."]}
```

```json
{"skills": {"all": {"0": ["\"min_years\" must be one of [2, 3, 4, 5]."]}}}
```

```json
{"non_field_errors": ["At least one term to search for is required."]}
```

```json
{"filters": {"seniority": {"any": {"0": ["\"lead\" is not a valid choice."]}}}}
```

#### Limit and access error format

```json
{"status": {"code": 1010, "message": "Request was throttled."}}
```

| `status.code` | HTTP | Meaning |
|---|---|---|
| `1010` | 429 | The daily limit of search results is exhausted |
| `1011` | 402 | Contacts / profiles quota is exhausted |
| `1012` | 402 | The Search API is not enabled for the company |
| `100` | 403 | GDPR: access to the profile is restricted |
| `102` | 403 | GDPR: the `age` filter is forbidden for your company |
| `103` | 403 | GDPR: the `gender` filter is forbidden |
| `104` | 403 | GDPR: the `diversity` filter is forbidden |

The `age`, `gender` and `diversity` filters are not available to every company — the GDPR settings of the account govern that. If you get `102`–`104`, drop the filter or discuss the settings with your manager.


### Search examples

#### Senior Go backend engineer, not from Google, speaks English

```json
{
  "skills": {
    "all": [{"value": "id-go", "min_years": 3}, "kubernetes"],
    "any": ["postgresql", "mysql"],
    "none": ["php"],
    "preferred": ["grpc"]
  },
  "last_positions": {"any": ["backend engineer", "backend developer"], "none": ["intern"]},
  "companies": {"none": ["id-google"]},
  "filters": {
    "seniority": {"any": ["senior", "team_lead"]},
    "experience_years": {"min": 5},
    "months_on_last_position": {"min": 12},
    "languages": [{"name": "English", "levels": ["full", "native"]}],
    "remote": true
  },
  "per_page": 100
}
```

#### Data scientist from top universities, with a phone, active on GitHub

```json
{
  "positions": {"any": ["data scientist", "ml engineer"]},
  "educations": {"any": ["id-mit", "id-stanford-university"]},
  "filters": {
    "contacts": {"any": ["phone"]},
    "education_levels": {"any": ["master", "doctorate"]},
    "sites": {"all": ["github.com"]}
  }
}
```

#### Everyone currently working at a given company

```json
{
  "last_companies": {"any": ["id-yandex"]},
  "per_page": 100
}
```

#### Estimate the volume before exporting

A request with `per_page: 1` spends one profile of the limit, while `count` returns the full size of the result set.

```json
{
  "skills": {"all": ["id-rust"]},
  "locations": {"any": ["id-germany"]},
  "per_page": 1
}
```


### Integration recommendations

- **Retry** only on `502`/`504` and network errors, with exponential backoff and a cap of 3–5 attempts. Do not retry `4xx`.
- **Log the body of `400` errors** — it points at the exact field.
- **Slug > text.** Where a slug is known, use it: a text match depends on the spelling in the source. Take slugs from `GET /suggestions/` and cache them — the dictionary changes rarely.
- **Check `count` after changing slugs**: a typo in `id-...` gives no error, it gives empty results.
- **Watch the schema** `GET /api/v6/schema/` on updates: new filters and values appear there first.

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
