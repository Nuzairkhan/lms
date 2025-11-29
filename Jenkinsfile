pipeline {
   agent any

   environment {
      SONAR_HOST_URL = "http://44.223.27.240:9000"
      SONAR_TOKEN = "squ_ef68f131502f5878f3a9c97d8e88c65100d5d0d5"
      NEXUS_URL = "http://44.223.27.240:8081/repository/lms/"
      NEXUS_USER = "admin"
      NEXUS_PASS = "lms12345"
   }

   stages {

       stage('Code Quality') {
           steps {
               echo 'Sonar Analysis Started'

               sh '''
                   docker run --rm \
                   -e SONAR_HOST_URL="${SONAR_HOST_URL}" \
                   -e SONAR_TOKEN="${SONAR_TOKEN}" \
                   -v "$WORKSPACE:/usr/src" \
                   sonarsource/sonar-scanner-cli \
                   -Dsonar.projectKey=lms
               '''

               echo 'Sonar Analysis Completed'
           }
       }

       stage('Build LMS') {
           steps {
               echo 'LMS Build Started'
               sh '''
                   cd webapp
                   npm install
                   npm run build
               '''
               echo 'LMS Build Completed'
           }
       }

       stage('Publish LMS') {
           steps {
               script {
                   def packageJson = readJSON file: 'webapp/package.json'
                   def version = packageJson.version

                   sh """
                       zip -r lms-${version}.zip webapp/build
                       curl -v -u ${NEXUS_USER}:${NEXUS_PASS} \
                       --upload-file lms-${version}.zip \
                       ${NEXUS_URL}
                   """
               }
           }
       }

       stage('Deploy LMS') {
           steps {
               script {
                   def packageJson = readJSON file: 'webapp/package.json'
                   def version = packageJson.version

                   sh """
                       curl -u ${NEXUS_USER}:${NEXUS_PASS} \
                       -X GET '${NEXUS_URL}lms-${version}.zip' \
                       --output lms-${version}.zip

                       sudo rm -rf /var/www/html/*
                       sudo unzip -o lms-${version}.zip
                       sudo cp -r webapp/build/* /var/www/html/
                   """
               }
           }
       }

       stage('Clean Up Workspace') {
           steps {
               cleanWs()
           }
       }
   }
}
