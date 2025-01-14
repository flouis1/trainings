# Lab: JFrog Security basics

## Goals

Understand Curation and JFrog Xray products

## Pre requisites

in your JFrog Project, create the following repositories :

Repo type | Repo key | Environment | Comment
---|---|---
REMOTE | <PROJECT_KEY>-npmjs-remote | DEV | for curation
VIRTUAL | <PROJECT_KEY>-npm  | DEV | include <PROJECT_KEY>-npmjs-remote
REMOTE | <PROJECT_KEY>-dockerhub-remote | DEV |
REMOTE | <PROJECT_KEY>-release-docker-local | DEV |
VIRTUAL | <PROJECT_KEY>-docker  | DEV | include the above repositories and set default deployment repo to PROJECT_KEY>-release-docker-local

## Curation

### Enable curation

Under "Administration" -> "Curation" -> "Curated Repositories", click the "State" toggle next to `<PROJECT_KEY>-npmjs-remote`

### Create a curation policy

Switch to the "Application" sidebar, and navigate to "Curation" -> "Policies Management", and click "Create New Policy".

   1. Policy Name: `malicious_packages`
   2. Repositories: "Specific", and then select `<PROJECT_KEY>-npmjs-remote`.
   3. Policy Condition: "Malicious package".
   4. Waivers: None (just click "Next").
   5. Actions: "Block". Also select "Notify by Email" and enter your email address.
   6. Click "Next", and then "Save Policy".

### Test the curation process

1. On your own machine, using the command prompt, navigate to the [common NodeJS module](../../common/js).
2. Configure your NPM client to download packages from your remote
    1. Artifactory > virtual repositories > <PROJECT_KEY>-npm
    2. Click on  `Setup Client` & follow the instruction
    3. Make sure in your ~/.npmrc that the ```_authToken``` is specified and NOT ```_auth```

    ```text
        //yann-sbx.jfrog.io/artifactory/api/npm/test-npm/:_authToken=cmVm*****
        always-auth=true
        email=yannc@jfrog.com
        registry=https://yann-sbx.jfrog.io/artifactory/api/npm/test-npm/
    ```

3. Try downloading a malicious package, such as `cors.js`:

   ```bash
   npm install cors.js
   ```

4. Check in the `<PROJECT_KEY>-npmjs-remote` repository that the cors.js package wasn't cached

## JFrog Xray

### Enable Xray indexing

1. Under "<PROJECT_NAME>" -> "Xray" -> "Indexed Resources", click the "Add repositories" and add all the created  repositories
2. For each repository, on the far right, click on `Actions` > `Configure`
3. Enable all the advanced scans

### Create a Xray security policy

1. Under "<PROJECT_NAME>" -> "Xray" -> "Watches & Policies", click on the "Policies" tab and then the "New policy" button
2. Set "security_policy" as the policy name and click on the "New Rule" button
3. Set "important" as the rule name
4. Set "Medium" as the severity level and save the rule

### Create a Xray watch

1. Under "<PROJECT_NAME>" -> "Xray" -> "Watches & Policies", click on the "Watches" tab and then the "Setup a watch" button
2. Set "docker" as the watch name
3. Add the docker repositories
4. Attach the "security_policy"

### Scan an artifact

1. Navigate to the [common NodeJS module](../../common/js).

    ```bash
    docker build <SAAS_DNS>/<PROJECT_NAME>-docker/my-js-demo:1.0.0
    ```

2. Confgure the docker client and push the image

    ```bash
    docker login <SAAS_DNS>
    docker push <SAAS_DNS>/<PROJECT_NAME>-docker/my-js-demo:1.0.0
    ```

3. Under "<PROJECT_NAME>" -> "Xray" -> "Scan List", check the scan results on the packages
