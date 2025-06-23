# PingCode Plugin

## About PingCode Plugin

 PingCode Plugin is an open source Jenkins plugin that can connect your builds and deployments with your Projects in PingCode. With this simple but powerful tool, you will keep updates about what happened on Jenkins, your builds and deployments, associated with your user stories, tasks and defects in real-time without leaving PingCode.

## Usage with PingCode

### Using PingCode `ID`

Using `#ID`in your commit messages, branch names and pull request titles, then the Jenkins plugin will automatically connect related builds and deployments when the job is running. As a result, team members will find the builds/deployments with related work items. `ID` is a unique identifier of a work item which can be found in PingCode at the top-left corner in its popup window.

| Category                              | Syntax                                                  | Example                                    |
| ------------------------------------- | ------------------------------------------------------- | ------------------------------------------ |
| Branch name                           | Supports bind to multiple `#ID` split "/".      | terry/#PLM-100/#PLM-101                    |
| Commit message and pull request title | Supports bind to multiple `#ID` split by space. | fix(doc): #PLM-100 #PLM-101 update the doc |

## Install

### Jenkins Marketplace

1. Login to your Jenkins server.
2. Navigate to the Plugin Manager.
3. Select the "Available" tab and search for `PingCode` as the plugin name then install it.
4. Restart jenkins for the install to take effect.

### Manual Install

1. Download pingcode.hpi from [pingcode jenkins plugins archive url](http://cdn.pingcode.com/jenkins/latest/pingcode.hpi).
2. Login to your Jenkins server.
3. Navigate to the Plugin Manager.
4. Select the "Advanced" tab and navigate to the Upload Plugin, upload pingcode.hpi file.
5. Restart jenkins for the install to take effect.

## Configure

### Create PingCode REST API App

1. Login to PingCode.
2. Go to "Admin console > Apps > Credentials" page.
3. Click "New".
4. Enter the following information:
   - App name.
   - Authentication method - `Client Credentials`.
   - Permission - The range of data that can be accessed. Give `DevOps data integration: Build` and `DevOps data integration: Delivery` read and write permission.
5. Copy Client ID and Client Secret.

### Configure Plugin

1. On the left navigation bar > `Manage Jenkins` > `Configure System` > `PingCode application`.
2. Enter the following information:
    - `Endpoint` - The URL of PingCode REST API. The default value is `https://open.pingcode.com`.
    - `Client id` - Copy from PingCode `Custom application` page (Client ID column).
    - `Client secret` - Click Add > Jenkins
      - For Kind, select `Secret text`.
      - For Secret, copy from PingCode `Custom application` page (Secret column).

      Once you add it successfully, you will find this Secret in the `Client secret` list and select it as a `Client secret`.

3. Click `Test Connection` to make sure your credentials are valid.

## Usage

The Jenkins plugin supports two styles of Jenkins items: `Freestyle project` and `pipeline`.

### Freestyle project

  1. Go into a specific Freestyle project in Jenkins.
  2. Find "Add post-build action" and click it.

  ![post.png](https://ftp.bmp.ovh/imgs/2020/08/5d033ad82a35cf5b.png)

##### Send build information

   1. Select `PingCode: create build record`.
   2. Enter the following information:

       - `Overview pattern` - Optional. A regular expression is used to match the result summary in the build output for display in PingCode.
         ![YqA1XQ.png](https://ftp.bmp.ovh/imgs/2020/08/3e86113c07f92196.png)

       - `Default summary` - Optional. If no information can be matched from the result logs, this value will be sent by default.

         ![default summary](https://ftp.bmp.ovh/imgs/2020/08/74a0b2938e370041.png)

      - `Result url` - Optional. A url of result for display in pingcode. if empty, the jenkins console result url will be used.
         ![result url](https://ftp.bmp.ovh/imgs/2020/08/ab6d2a256c19640f.png)

      - `Specified identifiers` - Optional. list of work item identifiers. for example `PLM-100,PLM-101`. If it is empty then the command will get the work item identifiers in the SCM.
               ![specified identifiers](https://s1.ax1x.com/2022/05/17/O4XHij.png)

  Finally, save these configurations. When the build is triggered, it will post the build information to PingCode. If there is a PingCode `#ID` in branch name, commit message or pull request title, you will get views in PingCode agile project about what happening on build.

##### Send deployment information

   1. Select `PingCode: create deploy record`.
   2. Enter the following information:

       - `Release name` - Required. The name of the release. You can use environment variables for dynamic variable substitution in the name. For example: `release-${BUILD_ID}`, which means that the release name is dynamically generated using the `BUILD_ID`. All environment variables injected by the plugin can be used. If the environment variable does not exist, the source character will be retained.
       - `Environment name` - Required. The name of environment that the code will be deployed to. If the environment does not exist, the plugin will automatically create.
       - `Release URL` - Optional. A URL that can view the detail deployment results. If it is empty, no related links are displayed in PingCode.
       - `Specified identifiers` - Optional. list of work item identifiers. for example `PLM-100,PLM-101`. If it is empty then the command will get the work item identifiers in the SCM.

       ![YbTMt0.png](https://s1.ax1x.com/2022/05/17/O4jKFH.png)

  Finally, save these configurations. When the deployment is triggered, it will post the deployment information to PingCode. If there is a PingCode `#ID` in branch name, commit message or pull request title, you will get views in PingCode agile project about what happening on deployment.

#### Pipeline Project

##### Send build information

  This is an example snippet of a very simple "build" stage set up in a Jenkinsfile. When the pipeline is triggered, it will post the build information to PingCode. If there is a PingCode `#ID` in branch name, commit message or pull request title, you will get views in PingCode agile project about what happening on build.

  ``` syntaxhighlighter-pre
    node {
        def summaryMessage = "Summary message"

        try {
            def scmVars = checkout scm
            sh "printenv"
            summaryMessage = "Great, build successfully."
        }
        catch(e) {
            echo e.getMessage()
            summaryMessage = e.getMessage()
        }
        finally{
            pingcodeBuildRecord(
                overviewPattern: "^JENKINS",
                defaultSummary: "${summaryMessage}",
                resultURL: "https://your-host/result-${env.BUILD_ID}",
                specifiedWorkItems: "PLM-100,PLM-101",
                failOnError: false
            )
        }
    }
  ```


  About `pingcodeBuildRecord`, you can get the following information:

- `overviewPattern` - Optional. A regular expression is used to match the result summary in the build result for display in PingCode.
- `defaultSummary` - Optional. If no information can be matched from the result logs, this value will be sent by default.
- `resultURL` - Optional. A URL that can view the detail results. If it is empty, no related links are displayed in PingCode.
- `workItemIdentifiers` - Optional. list of work item identifiers. If it is empty then the command will get the work item identifiers in the SCM.
- `failOnError` - Optional. When the value is true, if the process of sending build data to PingCode fails, the entire build will be marked as failed in Jenkins, otherwise Jenkins' build results will not be affected by it. The default value is false.

##### Send deployment information

  Below is an example of a very simple "deployment" stage set up in a Jenkinsfile. When the pipeline is triggered, it will post the deployment information to PingCode. If there is a PingCode `#ID` in branch name, commit message or pull request title, you will get views in PingCode agile project about what happening on deployment.

```syntaxhighlighter-pre
    node {
        try {
            def scmVars = checkout scm
            sh "printenv"
        }
        catch(e) {
            echo e.getMessage()
        }
        finally{
            pingcodeDeployRecord(
               releaseName: "release-${BUILD_ID}",
               environmentName: "Product",
               releaseURL: "https://www.pingcode.com/release-${JENKINS_HOME}",
               specifiedWorkItems: "PLM-100,PLM-101",
               failOnError: false
           )
        }
    }
  ```

  Ref `pingcodeDeployRecord`, you can get the following information:

- `releaseName`- Required. The name of the release. You can use environment variables for dynamic variable substitution in the name. For example: `release-${BUILD_ID}`, which means that the release name is dynamically generated using the `BUILD_ID`. All environment variables injected by the plugin can be used. If the environment variable does not exist, the source character will be retained.
- `environmentName` - Required. The name of environment that the code will be deployed to. If the environment does not exist, the plugin will automatically create.
- `releaseURL` - Optional. A URL that can view the detail deployment results. If it is empty, no related links are displayed in PingCode.
- `workItemIdentifiers` - Optional. list of work item identifiers. If it is empty then the command will get the work item identifiers in the SCM.
- `failOnError` - Optional. When the value is true, if the process of sending deployment data to PingCode fails, the entire deployment will be marked as failed in Jenkins, otherwise Jenkins' deployment results will not be affected by it. The default value is false.

## View Builds/Deployments in PingCode

Get views in PingCode agile project about what’s happening and insights with your Jenkins for things like:

- Build
- Deployment

![Y79yrV.png](https://s1.ax1x.com/2020/05/20/Y79yrV.png)

If you have any questions, please visit [https://pingcode.com/](https://pingcode.com/) and they will route it to the correct team to help you.
