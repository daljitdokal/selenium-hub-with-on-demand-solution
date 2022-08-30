# Selenium-hub

Selenium Grid is a testing tool which allows us to run our tests against different browsers.

This repository is centralised re-usable remote `gitlab` pipleine to run a version of [selenium](https://www.selenium.dev) utility.

## How to use

Following is the example of dynamic deployments and teardown of the `selenium-hub` and `node-chrome` pipeline. So that we only use the resources when required.

### Selenium hub and node chrome

```bash
stages:
  - Deploy Selenium
  - Deploy Chrome
  - Execute Tests
  - Uninstall Chrome
  - Uninstall Selenium
  - Publish Report

before_script:
  - export selenium_namespace=${openshift_namespace}
  - export application_name="my-app-name"
  - export route_domain="my-route-domain-name"

include:
  - project: 'selenium-hub-with-on-demand-solution'
    file: '/selenium/install-selenium-hub.gitlab-ci.yml'
  - project: 'selenium-hub-with-on-demand-solution'
    file: '/chrome/install-node-chrome.gitlab-ci.yml'
  - project: 'selenium-hub-with-on-demand-solution'
    file: '/chrome/uninstall-node-chrome.gitlab-ci.yml'
  - project: 'selenium-hub-with-on-demand-solution'
    file: '/selenium/uninstall-selenium-hub.gitlab-ci.yml'
  - project: 'selenium-hub-with-on-demand-solution'
    file: '/reporting/publish-report.gitlab-ci.yml'

# Deploy selenium grid
Deploy Selenium:
  stage: Deploy Selenium
  extends: .install_selenium_hub_template

# Deploy chrome node
Deploy Chrome:
  stage: Deploy Chrome
  extends: .install_chrome_template
  before_script:
  variables:
    selenium_chrome_replicas: 6

# Testing
Start testing:
  stage: Execute Tests
  script:
    - echo "my script here"

# Uninstall chrome node
Uninstall Chrome:
  stage: Uninstall Chrome
  extends: .uninstall_chrome_template

# Uninstall selenium grid
Uninstall Selenium:
  stage: Uninstall Selenium
  extends: .uninstall_selenium_hub_template

# Publish report
Publish Report:
  stage: Publish Report
  extends: .publish_report_template
```

### Set capability to use newly created node chromes

Update your `config-grid.js` file to let selenium know which chrome node to use.

```bash
capabilities: {
    browserName: 'chrome',
    applicationName: 'my-app-name', # ${application_name}
    ...

```
