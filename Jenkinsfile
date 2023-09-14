pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Sonar Analysis'
                sh 'cd frontend && sudo docker run  --rm -e SONAR_HOST_URL="http://52.91.131.78:9000" -e SONAR_LOGIN="sqp_b66c398a6b7e8aa919a23bb64db7c4796bfcaa7b"  -v ".:/usr/src" sonarsource/sonar-scanner-cli -Dsonar.projectKey=messageapp'
            }
        }    

        stage('Build & Release') {
            steps {
                script {
                def packageJSON = readJSON file: 'frontend/package.json'
                def packageJSONVersion = packageJSON.version
                sh "echo '${packageJSONVersion}'"

                echo 'Building'
                sh 'cd frontend && npm install && npm run build'
                
                echo 'Releasing'
                sh "zip frontend/dist-'${packageJSONVersion}'.zip -r frontend/dist"
                sh "curl -v -u admin:nexus --upload-file frontend/dist-'${packageJSONVersion}'.zip http://52.91.131.78:8081/repository/messaging-app-lms/"  
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                def packageJSON = readJSON file: 'frontend/package.json'
                def packageJSONVersion = packageJSON.version
                echo 'Deploying....'
                sh "curl -u admin:nexus -X GET \'http://52.91.131.78:8081/repository/messaging-app-lms/dist-${packageJSONVersion}.zip\' --output dist-'${packageJSONVersion}'.zip"
                sh 'sudo rm -rf /var/www/html/*'
                sh "sudo unzip -o dist-'${packageJSONVersion}'.zip"
                sh "sudo cp -r frontend/dist/* /var/www/html/"
                }
            }
        }
    }
}
