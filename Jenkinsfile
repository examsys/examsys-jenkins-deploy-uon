String libVersion = params.libVersion ?: 'production'
library "examsys-library@${libVersion}"

// Get the repository branches to use (default to the main branch used for development)
String examsysBranch = params.examsysBranch ?: 'develop'
String smsBranch = params.smsBranch ?: 'develop' // groovylint-disable-line DuplicateStringLiteral
String mappingBranch = params.mappingBranch ?: 'develop' // groovylint-disable-line DuplicateStringLiteral
String tinyPluginBranch = params.tinyPluginBranch ?: 'main'

// List of IP addresses that will be allowed into ExamSys when maintenance mode is enabled.
String maintenanceIPs = params.maintenanceIPs ?: ''
Boolean clean = params.clean ?: false

Boolean waitForConfirmation = params.waitForConfirmation ?: false
String confirmHook = params.confirmHook ?: ''

// This will store the name of the server that will be used to run the database upgrade.
String upgradeServer = params.upgradeServer
// This will be a list of all servers that will get the new code installed
String serverList = params.serverList
// The path on the servers that ExamSys should be installed into.
String path = params.deployPath ?: '/var/www/html'

// Colours used in Teams messages.
String noticeColour = '#3399ff'
String successColour = '#28a745'
String warningColour = '#ffc107'
String problemColour = '#dc3545'

// List of extra information we want to send with the notifications.
List fact = []

// Store the description as a fact.
if (params.buildDescription) {
    fact << [name: 'Build description', template: params.buildDescription]
}

// Build statuses.
Boolean cancelled = false
Boolean success = false

// Store the details about commit numbers.
String examsysCommit = ''
String smsCommit = ''
String mappingCommit = ''
String meeCommit = ''
String rubyCommit = ''

String message = ''
String colour = ''
String status = ''
String state = ''

String noMaintenance = 'Maintenance mode not enabled'

node {
    try {
        stage('Build') {
            dir('examsys') {
                examsysCommit = examsysBuild([
                    source: [url: params.examsysRepo, branch: examsysBranch, credentialsId: params.credentials],
                    maintenance: true,
                    maintenanceIPs: maintenanceIPs,
                    clean: clean
                ]) {
                    smsCommit = examsysAddPlugin([
                        type: 'SMS',
                        name: 'plugin_cs_sms',
                        source: [url: params.smsRepo, branch: smsBranch, credentialsId: params.credentials]
                    ])
                    mappingCommit = examsysAddPlugin([
                        type: 'mapping',
                        name: 'plugin_csmodule_mapping',
                        source: [url: params.mappingRepo, branch: mappingBranch, credentialsId: params.credentials]
                    ])
                    meeCommit = examsysAddTinymcePlugin([
                        name: 'maths-equation-editor',
                        source: [url: params.meeRepo, branch: tinyPluginBranch, credentialsId: params.credentials]
                    ])
                    rubyCommit = examsysAddTinymcePlugin([
                        name: 'ruby-annotation',
                        source: [url: params.rubyRepo, branch: tinyPluginBranch, credentialsId: params.credentials]
                    ])

                    examsysInstallLanguagePacks([version: examsysBranch])
                }

                fact << [name: 'ExamSys commit', template: examsysCommit]
                fact << [name: 'Campus Solutions SMS plugin commit', template: smsCommit]
                fact << [name: 'Campus Solutions mapping plugin commit', template: mappingCommit]
                fact << [name: 'Maths equation editor commit', template: meeCommit]
                fact << [name: 'Ruby annotations plugin commit', template: rubyCommit]
            }
        }

        stage('Await confirmation for downtime') {
            if (waitForConfirmation) {
                cancelled = true
                if (confirmHook) {
                    // We can let people know this is ready.
                    message = "Build $BUILD_NUMBER is waiting to put ExamSys into maintenance mode."
                    office365ConnectorSend color: noticeColour, message: message, webhookUrl: confirmHook
                }

                // Wait for approval.
                confirmedBy = input message: 'Continue deployment?', submitterParameter: 'confirmedBy'
                // Store who approved for the success notification.
                fact << [name: 'Downtime started by', template: confirmedBy]
                cancelled = false
            } else {
                echo 'Confirmation not required'
            }
        }

        stage('Maintenance mode on') {
            if (params.maintenanceMode) {
                examsysMaintenanceMode([
                    enable: true,
                    username: params.deployUsername,
                    path: path,
                    servers: serverList
                ])
            } else {
                echo noMaintenance
            }
        }

        stage('Deploy') {
            String archiveName = "$WORKSPACE/examsys/ExamSys-${examsysCommit}.tar.gz"
            String configDiff = examsysDeploy([
                deployFile: archiveName,
                servers: serverList,
                upgradeServer: upgradeServer,
                sshUser: params.deployUsername,
                credentialsId: params.deployCredentials,
                deployLocation: path,
                runUpgradeScript: params.runUpgradeScript,
                upgradeStaffHelp: params.upgradeStaffHelp,
                upgradeStudentHelp: params.upgradeStudentHelp
            ])

            if (configDiff) {
                fact << [name: 'Configuration changes', template: configDiff]
            }
        }

        stage('Maintenance mode off') {
            if (params.maintenanceMode) {
                examsysMaintenanceMode([
                    enable: false,
                    username: params.deployUsername,
                    credentialsId: params.deployCredentials,
                    path: path,
                    servers: serverList
                ])
            } else {
                echo noMaintenance
            }
            success = true
        }
    } finally {
        stage('Notifications') {
            if (cancelled) {
                message = "Build $BUILD_NUMBER was cancelled."
                colour = warningColour
                status = 'Cancelled'
                fact << [name: 'Cancelled by', template: confirmedBy]
                state = 'cancelled'
            } else if (success) {
                message = "Build $BUILD_NUMBER deployed successfully."
                colour = successColour
                status = 'Success'
                state = 'successful'
            } else {
                message = "Build $BUILD_NUMBER failed to deploy correctly."
                colour = problemColour
                status = 'Failure'
                state = 'failed'
            }

            if (params.jiraSite) {
                // Let JIRA know what happened to the build.
                jiraSendDeploymentInfo([
                    site: params.jiraSite,
                    environmentId: params.environmentID,
                    environmentName: params.environmentName,
                    environmentType: params.environmentType,
                    state: state
                ])
            }

            if (params.notifyHook) {
                office365ConnectorSend([
                    color: colour,
                    message: message,
                    factDefinitions: fact,
                    status: status,
                    webhookUrl: params.notifyHook
                ])
            } else {
                echo 'Notifications not required'
            }
        }
    }
}
