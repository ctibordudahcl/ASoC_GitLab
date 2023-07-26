# HCL AppScan on Cloud for GitLab CI/CD
Your code is better and more secure with HCL AppScan.

The HCL AppScan on Cloud for GitLab CI/CD enables you to run static analysis security testing (SAST) against the files in your repository and/or dynamic analysis security testint (DAST) against a running web application or REST APIs. The SAST scan identifies security vulnerabilities in your code and DAST scan in your deployed application and both store the results in AppScan on Cloud.

# Usage
## Register
If you don't have an account, register on [HCL AppScan on Cloud (ASoC)](https://www.hcltechsw.com/appscan/codesweep-for-github) to generate your API key and API secret.

## Setup
1. Generate your API key and API secret on [the API page](https://cloud.appscan.com/main/settings).
- The API key and API secret map to the `asoc_key` and `asoc_secret` parameters for this action. Store the API key and API secret as [secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) in your repository.
![adingkeys_animation](img/keyAndSecret.gif)
2. Create the application in ASoC. 
- The application ID in ASoC maps to application_id for this action.

# Required Inputs
| Name |   Description    |
|    :---:    |    :---:    |
| asocApiKeyId | Your API key from [the API page](https://cloud.appscan.com/main/settings) |
| asocApiKeySecret | Your API secret from [the API page](https://cloud.appscan.com/main/settings) |
| appId | The ID of the application in ASoC. |
| sevSecGw | Defines severity level for the security gateway (options: highIssues, mediumIssues, lowIssues or totalIssues). |
| maxIssuesAllowed | Maximum number of issues for selected severity level in variable sevSecGw. |

# Examples
```yaml
image: debian:latest

# The options to sevSecGw are highIssues, mediumIssues, lowIssues and totalIssues
# maxIssuesAllowed is the amount of issues in selected sevSecGw
# appId is application id located in ASoC 
variables:
  asocApiKeyId: xxxxxxxxxxxxxxxxxxx
  asocApiKeySecret: xxxxxxxxxxxxxxxxxxx
  appId: xxxxxxxxxxxxxxxxxxx
  sevSecGw: totalIssues
  maxIssuesAllowed: 200

stages:
- scan-sast

scan-job:
  stage: scan-sast
  script:
  # installing requeriments
  - 'apt update && apt install curl jq unzip -y'
  # Downloading and preparing SAClientUtil
  - curl https://cloud.appscan.com/api/SCX/StaticAnalyzer/SAClientUtil?os=linux > $HOME/SAClientUtil.zip
  - unzip $HOME/SAClientUtil.zip -d $HOME
  - rm -f $HOME/SAClientUtil.zip
  - mv $HOME/SAClientUtil.* $HOME/SAClientUtil
  - export PATH="$HOME/SAClientUtil/bin:${PATH}"
  # Generate IRX files based on source root folder downloaded by Gitlab
  - appscan.sh prepare
  # Authenticate in ASOC
  - appscan.sh api_login -u $asocApiKeyId -P $asocApiKeySecret -persist
  # Upload IRX file to ASOC to be analyzed and receive scanId
  - scanName=$CI_PROJECT_NAME-$CI_JOB_ID
  - appscan.sh queue_analysis -a $appId -n $scanName > output.txt
  - scanId=$(sed -n '2p' output.txt)
  - echo "The scan name is $scanName and scanId is $scanId"
```
