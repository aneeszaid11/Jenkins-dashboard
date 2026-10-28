import jenkins.model.*
import hudson.model.Job

pipeline {
    agent any

    stages {

        stage('Generate Build Dashboard') {
            steps {
                script {

                    // Change this to your top-level folder
                    def rootFolderName = "Github-Atlassian"

                    def rootFolder = Jenkins.instance.getItemByFullName(rootFolderName)

                    if (rootFolder == null) {
                        error("Folder not found: ${rootFolderName}")
                    }

                    def csvContent = "Job Name,Branch,Build Number,Status,Build Time\n"

                    def success = 0
                    def failed = 0
                    def total = 0

                    rootFolder.getAllItems(Job.class).each { job ->

                        total++

                        def build = job.getLastBuild()

                        if (build == null) {
                            csvContent += "\"${job.fullName}\",NO BUILDS,,,\n"
                            return
                        }

                        def branch = "N/A"

                        try {
                            def env = build.getEnvironment(null)

                            branch =
                                env["BRANCH_NAME"] ?:
                                env["GIT_BRANCH"] ?:
                                env["gitlabBranch"] ?:
                                env["SOURCE_BRANCH"] ?:
                                "N/A"

                        } catch(Exception ex) {
                            branch = "N/A"
                        }

                        def status = build.getResult()?.toString() ?: "RUNNING"

                        if (status == "SUCCESS") success++
                        if (status == "FAILURE") failed++

                        def buildTime = new Date(
                            build.getTimeInMillis()
                        ).format("yyyy-MM-dd HH:mm:ss")

                        csvContent += "\"${job.fullName}\"," +
                                      "\"${branch}\"," +
                                      "\"${build.number}\"," +
                                      "\"${status}\"," +
                                      "\"${buildTime}\"\n"
                    }

                    // Summary Row
                    csvContent += "\n"
                    csvContent += "\"TOTAL JOBS\",\"${total}\",,,\n"
                    csvContent += "\"SUCCESS\",\"${success}\",,,\n"
                    csvContent += "\"FAILED\",\"${failed}\",,,\n"

                    writeFile(
                        file: "Jenkins_Build_Dashboard.csv",
                        text: csvContent
                    )

                    echo "Dashboard generated successfully."
                }
            }
        }

        stage('Archive Report') {
            steps {
                archiveArtifacts(
                    artifacts: 'Jenkins_Build_Dashboard.csv',
                    fingerprint: true
                )
            }
        }
    }
}
