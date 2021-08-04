# ⚠️ This repository is deprecated.

GitHub has an official [OpenAPI Description](https://github.com/github/rest-api-description/) now

# Octokit routes

> machine-readable, always up-to-date GitHub REST API route specifications

![Update status](https://github.com/octokit/routes/workflows/Update/badge.svg)

## Downloads

- The de-referenced OpenAPI documents are [uploaded as attachments to each release](https://github.com/octokit/routes/releases)

Or install from package managers

- [npm](https://www.npmjs.com/package/@octokit/routes)
- [pypi](https://pypi.org/project/octokitpy-routes)

## Example

Example operation

```json
{
  "summary": "Lock an issue",
  "description": "Users with push access can lock an issue or pull request's conversation.\n\nNote that, if you choose not to pass any parameters, you'll need to set `Content-Length` to zero when calling out to this endpoint. For more information, see \"[HTTP verbs](https://developer.github.com/v3/#http-verbs).\"",
  "operationId": "issues-lock",
  "tags": ["issues"],
  "externalDocs": {
    "description": "API method documentation",
    "url": "https://developer.github.com/v3/issues/#lock-an-issue"
  },
  "parameters": [
    {
      "name": "accept",
      "description": "Setting to `application/vnd.github.v3+json` is recommended",
      "in": "header",
      "schema": {
        "type": "string",
        "default": "application/vnd.github.v3+json"
      }
    },
    {
      "name": "owner",
      "in": "path",
      "schema": {
        "type": "string"
      },
      "required": true,
      "description": "owner parameter"
    },
    {
      "name": "repo",
      "in": "path",
      "schema": {
        "type": "string"
      },
      "required": true,
      "description": "repo parameter"
    },
    {
      "name": "issue_number",
      "in": "path",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "issue_number parameter"
    }
  ],
  "requestBody": {
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "properties": {
            "lock_reason": {
              "description": "The reason for locking the issue or pull request conversation. Lock will fail if you don't use one of these reasons:  \n\\* `off-topic`  \n\\* `too heated`  \n\\* `resolved`  \n\\* `spam`",
              "type": "string",
              "enum": ["off-topic", "too heated", "resolved", "spam"]
            }
          }
        }
      }
    }
  },
  "responses": {
    "204": {
      "description": "Empty response"
    }
  },
  "x-code-samples": [
    {
      "lang": "Shell",
      "source": "curl \\\n  -XPUT \\\n  -H\"Accept: application/vnd.github.v3+json\" \\\n  https://api.github.com/repos/octocat/:repo/issues/:issue_number/lock"
    },
    {
      "lang": "JS",
      "source": "octokit.issues.get({\n  owner: 'octocat',\n  repo: 'hello-world',\n  issue_number: 1\n})"
    }
  ],
  "x-github": {
    "legacy": false,
    "enabledForApps": true,
    "githubCloudOnly": false
  },
  "x-changes": [
    {
      "type": "parameter",
      "date": "2019-04-10",
      "note": "\"number\" parameter renamed to \"issue_number\"",
      "meta": {
        "before": "number",
        "after": "issue_number"
      }
    }
  ]
}
```

Both endpoints or parameters may be deprecated. The `date` timestamp can be used to determine how long an Octokit library wants to support the endpoint / parameter. Deprecation information is located in the `x-changes` array of an operation.

Example for a deprecated parameter

```json
{
  "type": "parameter",
  "date": "2019-04-10",
  "note": "\"number\" parameter renamed to \"issue_number\"",
  "meta": {
    "before": "number",
    "after": "issue_number"
  }
}
```

Deprecated endpoints have a type of "idName"

```json
{
  "type": "idName",
  "date": "2019-03-05",
  "note": "\"List all licenses\" renamed to \"List commonly used licenses\"",
  "meta": {
    "before": {
      "idName": "list"
    },
    "after": {
      "idName": "list-commonly-used"
    }
  }
}
```

## Usage as Node module

```js
const ROUTES = require("@octokit/routes");
// ROUTES has "api.github.com" key and one "ghe-*" key for each supported GHE version
// The value of each key is the full OpenAPI specification for the respective version
```

## How it works

This package updates itself using a daily cronjob running on Travis. All [`openapi/*.json`](openapi/) files are generated by [`bin/octokit-rest-routes.js`](bin/octokit-rest-routes.js).

```bash
node bin/octokit-rest-routes.js update
```

Run `node bin/octokit-rest-routes.js --usage` for more instructions.

The update script is scraping [GitHub’s REST API](https://developer.github.com/v3/) documentation pages and extracts the meta information using [cheerio](https://www.npmjs.com/package/cheerio) and a ton of regular expressions :)

For simpler local testing and tracking of changes, all loaded pages are cached in the [`cache/`](cache/) folder. To only run the code generating the OpenAPI files without updating the cache, add the `--cached` option.

```bash
node bin/octokit-rest-routes.js update --cached
```

To update the enterprise routes for all versions, you have to set the `--ghe` option.

```bash
node bin/octokit-rest-routes.js update --ghe
```

You can optionally pass a version number

```bash
node bin/octokit-rest-routes.js update --ghe 2.20
```

### 1. Find documentation pages

- Index page cached in [`cache/api.github.com/v3/index.html`](cache/api.github.com/v3/index.html)
- Result cached in [`cache/api.github.com/pages.json`](cache/api.github.com/pages.json)

Opens https://developer.github.com/v3/, find all documentation page URLs in the side bar navigation.

### 2. On each documentation page, finds sections

- Documentation pages cached in `cache/v3/*/index.html`, e.g. [`cache/api.github.com/v3/repos/index.html`](cache/api.github.com/v3/repos/index.html)
- Documentation sub pages cached in `cache/v3/*/*/index.html`, e.g. [`cache/api.github.com/v3/repos/branches/index.html`](cache/api.github.com/v3/repos/branches/index.html)
- All sections found on pages are cached next to the `index.html` files, e.g. [`cache/api.github.com/v3/repos/sections.json`](cache/api.github.com/v3/repos/sections.json)
- HTML of sections are stored next to the `index.html` files, e.g. [`cache/api.github.com/v3/repos/create.html`](cache/api.github.com/v3/repos/create.html)

Loads HTML of each documentation page, finds sections in page.

### 3. In each section, finds endpoints

- Some sections don’t have endpoints, such as [Notifications Reasons](https://developer.github.com/v3/activity/notifications/#notification-reasons)
- Some sections have multiple endpoints, see [#3](https://github.com/octokit/routes/issues/3)

Loads HTML of documentation page section. Turns it into [`openapi/*.json`](openapi/) files. In some cases the HTML cannot be turned into an endpoint using the implemented patterns. For these cases [custom overrides](lib/endpoint/overrides) are defined.

## See also

- [octokit/graphql-schema](https://github.com/octokit/graphql-schema) – GitHub’s GraphQL Schema with validation
- [octokit/webhooks](https://github.com/octokit/webhooks) – GitHub Webhooks specifications

## LICENSE

[MIT](LICENSE.md)