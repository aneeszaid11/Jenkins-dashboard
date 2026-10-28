pipeline {
    agent any

    stages {
        stage('Generate Dashboard') {
            steps {
                script {

                    def jobs = [
                        "Github-Atlassian"
                    ]

                    def csv = "Job Name,Build Number,Status\n"

                    jobs.each { jobName ->

                        def response = bat(
                            script: """
                            curl -s "${JENKINS_URL}/job/${jobName}/lastBuild/api/json"
                            """,
                            returnStdout: true
                        ).trim()

                        def json = readJSON text: response

                        csv += "${jobName},${json.number},${json.result}\n"
                    }

                    writeFile file: "dashboard.csv", text: csv

                    archiveArtifacts artifacts: "dashboard.csv"
                }
            }
        }
    }
}
