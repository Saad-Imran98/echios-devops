# DevOps essentials Tools

DevOps essentials tools: Nexus for artifact management, SonarQube for code quality, Jira for project tracking, and Jenkins for continuous integration and automation.

## Jira 
- Login / Signup
  - [https://id.atlassian.com/login]
- Create
  - Project
- Permission to delete issue
  - Project > Setttings > Permissions
    - Actions > Edit permissions
    - Search "Delete Issues" > Update > Any Logged in User
  - Start Docker services in codespaces
  - Create
    - Project and issues
## GitHub codespaces
- Create a Github repo with readme.md
- From Codespaces > Create codespaces > choose above repo > select 4 core/16GB RAM create codespace.
- Open codespace in Browser
  - Either Option 1 : In Terminal
  ```
  git clone https://github.com/ckechios/devops-tools-bootstrap-starter.git
  ```
  - In Codespaces : File Explorer
    - In Codespaces explorer > Drag all from the directory devops-tools-bootstrap-starter to root
    - Select directory devops-tools-bootstrap-starter > Delete directory
  - Or Option 2 : In Terminal
  ```
  git remote add starter https://github.com/ckechios/devops-tools-bootstrap-starter.git
  git fetch starter 
  git merge starter/main --allow-unrelated-histories 
  git remote remove starter 
  ```

## Docker
- run tools with docker compose up -d
- Commands for docker
```
docker ps
docker ps -a
docker compose up -d   # run in detached mode
docker compose down
docker logs [containerid]
docker exec -it [containerid] bash
```

## Jira
- Create project
- Show Epic, Story, Task
- Show Apps > Zephyr
- Jira API key
  - Jira Proflie > manage Account : Security
    - API tokens : Create and manage API tokens
    - Create API token - Create token name: jira-jenkins
    - copy and paste in .creds file

## Maven
- Maven commands
```
mvn clean install
mvn clean package
mvn deploy package   # For Nexus deployment

```

## Nexus setup for local development and pushing to repo
- Port is 8081 : Make port public
- To get nexus password
  - > docker exec -it [nexus-conatiner-id] cat [get filename from login screen]
## Nexus - create artifact repository
- Setup maven repository
  - Release - name= echart
  - Snapshot - name= echop
    - Allow redeploy
- Deploy manually using Maven
  - mvn deploy package
- Direct uplaod using mvn update
- Set username and pass for nexus repo
- This should match <distributionManagement> <repository> <id> in POM
- > code ~/.maven/current/conf/settings.xml
  - with code below
  - ```
    <!-- between the <servers> tag -->
    <server>
      <id>nexus-snapshots</id>
      <username>admin</username>
      <password>pass</password>
    </server>
    
    <server>
      <id>nexus-releases</id>
      <username>admin</username>
      <password>pass</password>
    </server>
    ```
  - Check pom.xml update the <distributionManagement> section
  -
    ```
    <distributionManagement>
        <repository>
            <id>nexus-releases</id>   <!-- must correspond with ~/.maven/current/conf/settings.xml id -->
            <!-- nexus url for artifact/snapshot repo -->
            <url>https://YOURGITHUB_NEXUS_URL-8081.app.github.dev/repository/echart/</url>
        </repository>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>https://YOURGITHUB_NEXUS_URL-8081.app.github.dev/repository/echop/</url>
        </snapshotRepository>
    </distributionManagement>
    ```
  - Go into calculator-module - build and push to Nexus repo
    - mvn deploy package - Ensure ~/.maven/current/conf/settings.xml has the correct credentials

## SonarQube setup
- Generate SonarQube Token
  - sonarqube UI - My Account > Security > Generate Token : User Token : Generate - name: sonar-scan
    - SonarQube key looks like this: squ_b64682b7e2569ac6fe6edf05cda81abc59d8b846
- mention how -Dsona.coverage is for not covering test cases - else sonar will fail

## scanning with sonar locally
- Port is 9000 : Make port Public
- initial login credentials, admin : pass
```
export SONAR_TOKEN=YOUR_SONAR_TOKEN
#sonar_token set in env like below

export SONAR_TOKEN=squ_[randomcharacters]

mvn clean verify sonar:sonar -Dsonar.host.url=https://YOUR_GITHUB_SONARQUBE_URL-9000.app.github.dev -Dsonar.coverage.exclusions=**/*
```

## Jira API
- Set up API
  - Profile Picture > Manage account : Security > API Tokens > Create and manage API tokens
  - jira-jenkins : User Name and Password (where pass is key, user is email, id is: jira-jenkins)
- Test connection
```
  curl -X GET -u <email>:<token> -H "Content-Type: application/json" -H "Accept: application/json" https://yourcloud.atlassian.net/rest/api/2/issue/ISSUE_NUM
```

## Jenkins setup
- initial password 
  - > docker logs [jenkins container id]
  - > or check inside container
     - > docker exec -it jenkins bash
     - > cat /var/jenkins_home/secrets/initialAdminPassword
    - or
    - ```
      docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
      ``` 
- Plugins
  - nexus plugin :
    - Nexus Artifact Uploader Version 2.14
    - Pipeline Utility Steps
  - SonarQube Scanner
  - JIRA (just Jira plugin - need to restart server - need to restart docker)
- Setup credentials
  - jira-jenkins : User Name and Password (where pass is key, user is email, id is: jira-jenkins)
  - sonar-scan : Secret text
  - nexus-jenkins : User Name and Password
- Setup Tools :
  - Manage Jenkins
    - Manage Jenkins > System
      - SonarQube servers
        - Environment Varaibles MUST be checked
        - Add SonarQube
          - Name: sonarserver
          - URL: https://GITHUB_URL-9000.app.github.dev
          - server authentication token
      - Jira : click + Add
        - URL : https://YOUR_DOMAIN.atlassian.net/
        - CREDENTIALS : jira-jenkins
    - Manage Jenkins > Tools
      - Add Maven
        - Name M3
        - version 3.9.9

## Jenkins stages
- MVN build stage
  - ```
    stage('Build') {
      steps {
        dir('calculator') {
          sh 'cat src/main/java/com/echios/App.java '
          sh 'mvn clean install'
        }
      }
    }
    ```

- Declare Environment after tools { maven 'M3' }
- ```
  environment {
      pom = readMavenPom(file: 'calculator/pom.xml')
      artifactId = pom.getArtifactId()
      version = pom.getVersion()
      name = pom.getName()
      groupId = pom.getGroupId()
  }
  ```
- Environment variable Check stage
  - ```
    stage('Environment details') {
      steps {
        dir('calculator') {
          script {
            echo "Version: ${version}"
            echo "Group ID: ${groupId}"
            echo "Artifact ID: ${artifactId}"
          }
        }
      }
    }
    ```
  - #### NOTES : On Environment variable check pipeline
    - Build fails because of variable approval
    - Manage Jenkins > In-process Script Approval > Approve
    - **Keep approving for each variable** like group id, artifact id, version, name
    - 
- Sonar analysis stage
  - ```
    stage('SonarQube Analysis...') {
        steps {
            dir('calculator') {
          script {
            withSonarQubeEnv(credentialsId: 'sonar-scan') {
              sh 'mvn clean package sonar:sonar -Dsonar.coverage.exclusions=**/*'
            }
          }
            }
        }
    }
    ```
- Push to Nexus Snapshot stage
  - Before pasting the script learn how to get the details
  - Click on Pipeline sample - Pipeline Project page
    - sample step of nexusArtifactUploader
    - enter details from pom.xml and nexus repo
    - Generate PIpeline Script
  - ```
    stage('Send to Nexus snapshot rep') {
        steps {
            dir('calculator') {
              echo 'Nexus upload'
              nexusArtifactUploader(
                credentialsId: 'nexus-jenkins',
                groupId: "${groupId}",
                nexusUrl: 'YOURGITHUB_NEXUS_URL-8081.app.github.dev',
                nexusVersion: 'nexus3',
                protocol: 'https',
                repository: 'echop',
                version: "${version}",
                artifacts: [
                    [
                        artifactId: "${artifactId}",
                        classifier: '',
                        file: "target/${artifactId}-${version}.jar",
                        type: 'jar'
                    ]
                ]
              )
            }
        }
    }    
    ```
  - #### NOTES : On Nexus pipeline
    - Build fails because of variable approval
    - Manage Jenkins > In-process Script Approval > Approve
    - **Keep approving for each variable** like group id, artifact id, version, name

- JIRA update stage
  - ```
    stage('JiraUpdate') {
        steps {
            jiraComment body: "Sonar Passed & pushed to Nexus (Job: ${env.JOB_NAME}, Build: ${env.BUILD_NUMBER})", issueKey: 'your_issue_key'
        }
    }
    ```

