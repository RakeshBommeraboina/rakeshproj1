def registry='https://trialvxgmj3.jfrog.io/'
pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {

        stage("git clone") {
            steps {
                git url: 'https://github.com/RakeshBommeraboina/sparkjava-war.git',
                    branch: 'main'
            }
        }

        stage('build') {
            steps {
                sh 'mvn clean deploy'
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'sonarqube-scanner'
            }

            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("Jar Publish") {
            steps {
                script {
                    echo '<-------------- Jar Publish Started -------------->'

                    def server = Artifactory.newServer(
                        url: 'registry + "/artifactory"',
                        credentialsId: "artifact-credentials"
                    )

                    def properties = "build=${env.BUILD_ID},commitId=${GIT_COMMIT}"

                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "rakesh-libs-snapshot-local",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""

                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)

                    echo '<-------------- Jar Publish Ended -------------->'
                }
            }
        }
    }
}
