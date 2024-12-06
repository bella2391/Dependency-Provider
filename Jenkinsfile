pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh './gradlew build'  // または ./mvnw package などでJARをビルド
            }
        }
        stage('Configure Git') {
            steps {
                withCredentials([
                    string(credentialsId: 'GitUserName', variable: 'GIT_USER_NAME'),
                    string(credentialsId: 'GitUserEmail', variable: 'GIT_USER_EMAIL')
                ]) {
                    sh '''
                    pwd
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
                        # リポジトリが既に存在している場合はクローンしない
                        if [ ! -d "Jenkin-Dependency-Provider" ]; then
                            git clone "https://github.com/bella2391/Jenkin-Dependency-Provider.git"
                        fi
                        cd Jenkin-Dependency-Provider
                        git tag -a "jenkins-FMC-Dependency-${BUILD_NUMBER}" -m "Jenkins Build #${BUILD_NUMBER}"
                        git push "https://oauth2:${GIT_TOKEN}@github.com/bella2391/Jenkin-Dependency-Provider.git" --tags
                        '''
                    }
                }
            }
        }
        stage('Create Release on GitHub') {
            steps {
                script {
                    // GitHub APIを使用してリリースを作成するステップ
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
                    echo "Release created: ${response}"
                }
            }
        }
        stage('Upload JAR to Release') {
            steps {
                script {
                    // GitHubリリースにJARファイルをアップロード
                    def releaseUrl = "https://uploads.github.com/repos/bella2391/Jenkin-Dependency-Provider/releases/${release_id}/assets?name=${file_name}"
                    sh "curl -XPOST -H 'Authorization: token ${GIT_TOKEN}' -H 'Content-Type: application/octet-stream' --data-binary @path_to_jar_file ${releaseUrl}"
                }
            }
        }
    }
}
