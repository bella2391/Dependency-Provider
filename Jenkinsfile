pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh './gradlew build'
            }
        }
        stage('Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh './gradlew test'
                }
            }
        }
        stage('Configure Git') {
            steps {
                withCredentials([
                    string(credentialsId: 'GitUserName', variable: 'GIT_USER_NAME'),
                    string(credentialsId: 'GitUserEmail', variable: 'GIT_USER_EMAIL')
                ]) {
                    sh '''
                    git config --global user.name "${GIT_USER_NAME}"
                    git config --global user.email "${GIT_USER_EMAIL}"
                    '''
                }
            }
        }
        stage('Tag and Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'GITACCESSTOKEN', variable: 'GIT_TOKEN')]) {
                        sh '''
                        if [ -d "Jenkin-Dependency-Provider" ]; then
                            rm -rf Jenkin-Dependency-Provider
                        fi
                        git clone "https://github.com/bella2391/Jenkin-Dependency-Provider.git"
                        cd Jenkin-Dependency-Provider
                        if git rev-parse "refs/tags/jenkins-FMC-Dependency-${BUILD_NUMBER}" >/dev/null 2>&1; then
                            echo "Tag jenkins-FMC-Dependency-${BUILD_NUMBER} already exists. Skipping tag creation."
                        else
                            git tag -a "jenkins-FMC-Dependency-${BUILD_NUMBER}" -m "Jenkins Build #${BUILD_NUMBER}"
                            git push "https://oauth2:${GIT_TOKEN}@github.com/bella2391/Jenkin-Dependency-Provider.git" --tags
                        fi
                        '''
                    }
                }
            }
        }
        stage('Create Release on GitHub') {
            steps {
                script {
                    def response = httpRequest(
                        acceptType: 'APPLICATION_JSON',
                        contentType: 'APPLICATION_JSON',
                        httpMode: 'POST',
                        url: 'https://api.github.com/repos/bella2391/Jenkin-Dependency-Provider/releases',
                        requestBody: """
                        {
                            "tag_name": "jenkins-FMC-Dependency-${BUILD_NUMBER}",
                            "name": "Release ${BUILD_NUMBER}",
                            "body": "Jenkins build release for build ${BUILD_NUMBER}",
                            "draft": false,
                            "prerelease": false
                        }
                        """,
                        authentication: 'GitHub-Token'
                    )
                    echo "GitHub API Response: ${response.content}"
                    def release_id = new groovy.json.JsonSlurper().parseText(response.content).id
                    def jarFilePath = 'build/libs/FMC-Dependency-1.0.0.jar'
                    def fileName = 'FMC-Dependency-1.0.0.jar'
                    sh '''
                    if [ ! -f "${jarFilePath}" ]; then
                        echo "Error: JAR file not found at ${jarFilePath}"
                        exit 1
                    fi
                    '''
                    sh "curl -X POST -H 'Authorization: token ${GIT_TOKEN}' -H 'Content-Type: application/octet-stream' --data-binary @${jarFilePath} https://uploads.github.com/repos/bella2391/Jenkin-Dependency-Provider/releases/${release_id}/assets?name=${fileName}"
                }
            }
        }
    }
}
