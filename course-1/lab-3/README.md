# Lab: JFrog BOMs basics

## Goals

Practice manipulating JFrog BOMs

## Create a repository structure

in your JFrog Project, create the following repositories :

Repo type | Repo key | Environment | Comment
---|---|--- |---
LOCAL | <PROJECT_KEY>-rc-maven-local | DEV |
LOCAL | <PROJECT_KEY>-release-maven-local | PROD |
REMOTE | <PROJECT_KEY>-mavencentral-remote | DEV |
VIRTUAL | <PROJECT_KEY>-maven  | DEV | include the 3 repos above and set default deployement to  <PROJECT_KEY>-rc-maven-local

### Create build info (only from API)

1. Navigate to [shared Java project directory](../../common/java).
2. Create build configuration:

   ```bash
   export MY_PROJ_KEY=<PROJECT_KEY>
   jf mvnc \
      --repo-deploy-releases $MY_PROJ_KEY-maven \
      --repo-deploy-snapshots $MY_PROJ_KEY-maven \
      --repo-resolve-releases $MY_PROJ_KEY-maven \
      --repo-resolve-snapshots $MY_PROJ_KEY-maven
   ```

3. Build & deploy:

   ```bash
   export JFROG_CLI_BUILD_NAME=$MY_PROJ_KEY-app  JFROG_CLI_BUILD_NUMBER=1
   jf mvn clean package deploy 
   ```

4. Push the build info to Artifactory:

   ```bash
   jf rt bp
   ```

Then, navigate to "Artifactory" -> "Builds", and show `green-app` and its build (`1`).
Show the Build info repository on the UI

### Create RBv2 (from UI)

Show the existing Release Bundle repository on the UI

1. From the build's screen in Artifactory, click on a build name 
2. Hover over a version and click on the 3 dots on the far right
3. Click on "Create Release Bundle".

> You can also create a Release Bundle at the Build Version level

* Release Bundle Name: `<PROJECT_KEY>-release`
* Release Bundle Version: `1.0`
* Signing Key: `main`

Click "Next", then "Create".

### Promote RBv2

> The order of environment can be changed on the UI (Plaform configuration > Environments)

After navigating to the RBv2's screen, click "Promote".

* For Signing Key, select `main`.
* For Target Environment, select `PROD`.

Click "Next", ensure that the "Target Repositories" for Maven artifacts is set properly, and click "Promote".
