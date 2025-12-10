pipeline {
    agent any

    stages {

        stage('Code Quality') {
            steps {
                echo 'Sonar Analysis Started'
                sh '''
                    cd webapp
                    docker run --rm \
                    -e SONAR_HOST_URL="http://54.172.35.60:9000" \
                    -v $(pwd):/usr/src \
                    -e SONAR_TOKEN="squ_fc270392a4b8b73b2315b5aee3ff627480d2073f" \
                    sonarsource/sonar-scanner-cli \
                    -Dsonar.projectKey=lms
                '''
                echo 'Sonar Analysis Completed'
            }
        }

        stage('Build LMS') {
            steps {
                echo 'LMS Build Started'
                sh 'cd webapp && npm install --no-progress --verbose && npm run build'
                echo 'LMS Build Completed'
            }
        }

        stage('Publish LMS') {
            steps {
                script {
                    def pkg = readJSON file: 'webapp/package.json'
                    def ver = pkg.version
                    echo "Version: ${ver}"

                    // Create ZIP file
                    sh "zip -r webapp/lms-${ver}.zip webapp/dist"

                    // Upload file to Nexus (FIXED PATH + FIXED URL)
                    sh """
                    curl -v -u admin:lms12345 \
                    --upload-file ${WORKSPACE}/webapp/lms-${ver}.zip \
                    http://54.172.35.60:8081/repository/lms/lms-${ver}.zip
                    """
                }
            }
        }

        stage('Deploy LMS') {
            steps {
                script {
                    def pkg = readJSON file: 'webapp/package.json'
                    def ver = pkg.version
                    echo "Deploying ${ver}"

                    // Download artifact from Nexus
                    sh """
                    curl -u admin:lms12345 -X GET \
                    http://54.172.35.60:8081/repository/lms/lms-${ver}.zip \
                    --output lms-${ver}.zip
                    """

                    // Clear old files
                    sh "rm -rf /var/www/html/*"

                    // Unzip downloaded build
                    sh "unzip -o lms-${ver}.zip"

                    // Copy final build to web root
                    sh "cp -r webapp/dist/* /var/www/html"
                }
            }
        }

        stage('Clean Up Workspace') {
            steps {
                echo 'Cleaning Workspace'
                cleanWs()
            }
        }
    }
}
