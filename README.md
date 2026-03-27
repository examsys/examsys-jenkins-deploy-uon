# University of Nottingham build

This is a pipeline that will build and deploy ExamSys with the University of Nottingham plugins installed

Requires that the [ExamSys global trusted pipeline library](https://github.com/examsys/examsys-jenkins-library)
is installed.

The script requires that ExamSys has already been installed on all the servers it is operating on because we need the
existing config file from the installation.

It will also fail if any of the front ends have a previous partial deployment on them.

## Variables

| Variable            | Required           | Explanation                                                                                                      |
|---------------------|--------------------|------------------------------------------------------------------------------------------------------------------|
| buildDescription    | No                 | A description of the build being deployed                                                                        |
| confirmHook         | No                 | The MS Teams webhook url to post a confirmation required message to                                              |
| credentials         | Yes                | The id of a set of stored credentials (SSH username and private key) that will authenticate the git repositories |
| deployCredentials   | Yes                | The credentials (username and password) that are used to connect to the ExamSys database on the upgrade server   |
| deployUsername      | Yes                | The username that is used to deploy code on the servers                                                          |
| deployPath          | No                 | The base installation path of ExamSys (default: /var/www/html)                                                   | 
| environmentID       | If jiraSite is set | The id of the environment we are deploying to                                                                    |
| environmentName     | If jiraSite is set | The name of the environment we are deploying to                                                                  |
| environmentType     | If jiraSite is set | One of: unmapped, development, testing, staging, production                                                      |
| examsysRepo         | Yes                | The ExamSys repository                                                                                           |
| examsysBranch       | Yes                | The branch of ExamSys to checkout                                                                                |
| jiraSite            | No                 | The URL of a JIRA site                                                                                           |
| libVersion          | No                 | The version of the ExamSys global trusted pipeline to use (default: production)                                  |
| maintenanceIPs      | No                 | List of IP addresses that should be allowed to access ExamSys during a maintenance window                        |
| maintenanceMode     | Yes                | Flags if maintenance mode should be enabled during the deployment                                                |
| mappingRepo         | Yes                | The repository for the mapping plugin                                                                            |
| mappingBranch       | No                 | The version of the mapping plugin to checkout (default: develop)                                                 |
| meeRepo             | Yes                | The repository for the maths equation editor TinyMCE plugin                                                      |
| notifyHook          | No                 | The MS Teams webhook for Notifying about the overall build status                                                |
| readOnlyServerList  | No                 | List of servers that should be set to have a read only file system                                               |
| rubyRepo            | Yes                | The repository for the ruby annotations TinyMCE plugin                                                           |
| runUpgradeScript    | Yes                | Flag if the database upgrade should run                                                                          |
| serverList          | Yes                | The list of servers that ExamSys code should be deployed to                                                      |
| smsRepo             | Yes                | The repository for the SMS plugin                                                                                |
| smsBranch           | No                 | The version of the SMS plugin to checkout (default: develop)                                                     |
| tinyPluginBranch    | No                 | The branch to be used in the TinyMCE plugins (defaul: main)                                                      |
| upgradeServer       | Yes                | The ExamSys server that the database upgrade should take place on                                                | 
| upgradeStaffHelp    | Yes                | Flag if the staff help should be updated                                                                         |
| upgradeStudentHelp  | Yes                | Flag if the student help should be updated                                                                       |
| waitForConfirmation | No                 | If the job should wait for confirmation before deployment (default: false)                                       |

## Required Jenkins plugins

* [Git](https://plugins.jenkins.io/git/)
* [Office-365-Connector](https://plugins.jenkins.io/Office-365-Connector/)

## Development of the pipeline

The library has three branches:

* **production** - This branch should be stable and tested, it will be used for jobs that are used in production.
* **testing** - This branch should be used during testing to ensure that future releases of the code work.
* **development** - New changes should go into this branch

The project can be linted on Linux using:

```bash
bin/lint
```
