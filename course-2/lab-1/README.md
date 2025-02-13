# Lab: Artifact management basics

## Goals

Perform basics actions via the UI & API regarding artifacts management

## Create a generic repository using the UI

Name it `<USERNAME>-test-generic-local`

## Upload / Download via the REST API

1. Upload a random file

   ```bash
      curl \
         -X PUT \
         -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
         -d "@test.txt" \
      $JFROG_SAAS_URL/artifactory/<USERNAME>-test-generic-local/test.txt
   ```

2. Delete the file from your local machine.
3. Download the file using the REST API:

   ```bash
      curl \
         -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
      $JFROG_SAAS_DNS/artifactory/<USERNAME>-test-generic-local/test.txt
   ```

## Upload / Download via the JFrog CLI

1. Open a command-line window, and browse to the directory containing your file.
2. Upload the file to the repository using the JFrog CLI:

   ```bash
   jf rt upload "*.txt" <USERNAME>-test-generic-local/cli-tests/
   ```

3. Download the file from the repository into your local machine:

   ```bash
   jf rt download <USERNAME>-test-generic-local/cli-tests/ .
   ```

## Apply properties via the UI

1. In the artifacts browser view, navigate to the file you just uploaded.
2. Navigate to the `Properties` tab.
3. Add the following properties :
   + `app.name` with the value `snake`
   + `app.version` with the value `1.0.0`

## Apply properties via the REST API

Assign the following properties to a file

```bash
   curl \
      -X PUT \
      -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
   "$JFROG_SAAS_URL/artifactory/api/storage/<USERNAME>-test-generic-local/test1.txt?properties=os=win,linux;qa=done"
```

## Apply properties via the JFrog CLI

Assign the following properties to a file

+ runtime.deploy.datetime=20240219_08000
+ runtime.deploy.account=robot_sa

```bash
jf rt sp "runtime.deploy.datetime=20240219_08000;runtime.deploy.account=robot_sa" <USERNAME>-test-generic-local/cli-tests/test2.txt .
```

## Search for artifacts with Artifactory Query Language (AQL)

> Here is the [official documentation for AQL](https://jfrog.com/help/r/jfrog-rest-apis/artifactory-query-language)

```bash
# we supposed we are in jfrog-training/course-1/lab-1
jf rt curl -XPOST -H "Content-type: text/plain" api/search/aql -d"@../../demos/basics-search/query-aql-properties-rest.txt"

jf rt s --spec="../../demos/basics-search/query-aql-cli.json"
```

## Search for artifacts with GraphQL

> Here is the [official documentation for GraphQL](https://jfrog.com/help/r/jfrog-rest-apis/graphql)

```bash
# the JFrog CLI rt curl command doesn't target metadata/api
# we have to use curl
curl \
    -XPOST \
    -H "Content-Type: application/json" \
    -d "@../../demos/basics-search/query-graphql.json" \
$JFROG_SAAS_DNS/metadata/api/v1/query 
```

### GraphiQL

1. In your browser, go to  `$JFROG_SAAS_DNS/metadata/api/v1/query/graphiql`and specify your access token
2. Extract the query from the JSON file  '{"query" : "<QUERY_TO_EXTRACT>"}  from `../../demos/basics-search/query-graphql.json`
3. Paste it in the query editor and execute it

## Create permission targets via the API

> **IMPORTANT NOTE** : From Artifactory V7.72.0, the permission targets are managed by JFrog Access (internal microservice) and so the official API endpoint is ```access/api/v2/permissions```. The previous API endpoint ```artifactory/api/v2/security/permissions``` is still maintained for the moment but will be deprecated in the future (no ETA).

Create the following permission target(s) :

Permission name | Resources | Population | Action | Comment
---|---|--- |--- |---
developers | All Remote  | developers group | Read, Deploy/Cache
uploaders  | All Remote + All local | uploaders group | Read, Deploy/Cache, Delete/Overwrite

> Here is the [official documentation on the API](https://jfrog.com/help/r/jfrog-rest-apis/permissions)

```bash
curl \
   -X POST \
   -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
   -H "Content-Type: application/json" \
   -d @"../../demos/advanced-access-authorization/payload/pt-api-def-latest.json" \
$JFROG_SAAS_URL/access/api/v2/permissions/
```

## Create permission targets via the JFrog CLI

> relies on [```artifactory/api/v2/security/permissions```](https://jfrog.com/help/r/jfrog-rest-apis/create-permission-target)

Create the following permission target(s) :

Permission name | Resources | Population | Action | Comment
---|---|--- |--- |---
consumers  | All Remote + All local | deployers group | Read, Annotate

```bash
# generate 1 permission target definition and store it into permissions.json
jf rt ptt pt-cli-template.json

# apply 1 permission target definition
jf rt ptc --vars pt-cli-template.json
```

## Creating Scoped Tokens

> Here is the [official documentation for Tokens](https://jfrog.com/help/r/jfrog-rest-apis/access-tokens)

### Identity token

Generate an identity token (will inherit the permission related to the current user)

```bash
curl \
   -XPOST \
   -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
   -d "scope=applied-permissions/user" \
$JFROG_SAAS_URL/access/api/v1/tokens
```

### Scoped token

Generate a token based on groups (will inherit the permission related to the groups)

```bash
curl \
   -XPOST \
   -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
   -d "scope=applied-permissions/groups:deployers" \
$JFROG_SAAS_URL/access/api/v1/tokens
```

### Scoped token for a transient user (non existing user)

> a token can be [refreshed](https://jfrog.com/help/r/jfrog-rest-apis/refresh-token)

Generate a transient user (will inherit the permission related to the specified groups)

```bash
# the token will expire in 300 seconds and can be refreshed
# it has to be executed by an Admin
curl \
   -XPOST \
   -H "Authorization: Bearer $JFROG_ACCESS_TOKEN" \
   -d "username=ninja" \
   -d "refreshable=true" \
   -d "expires_in=300" \
   -d "scope=applied-permissions/groups:developers" \
$JFROG_SAAS_URL/access/api/v1/tokens
```
