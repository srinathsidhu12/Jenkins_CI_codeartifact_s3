pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        CODEARTIFACT_DOMAIN = 'app-domain'
        CODEARTIFACT_REPO = 'sample_spring_boot_app_repo'
        ACCOUNT_ID = '654654304213'
        S3_BUCKET = 'spring-boot-app-build-artifacts'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                //Fetch application code
                git branch: 'master',
                    url: 'https://github.com/srinathsidhu12/Jenkins_CI_codeartifact_s3.git'
            }
        }
        stage('Debug AWS CLI') {
           steps {
               sh '''
               which aws
               aws --version
              /usr/bin/aws --version
              /usr/bin/aws codeartifact login --help
              '''
           }
        }         
        stage('Authenticate to CodeArtifact') {
            steps {
                //Fetches auth token
                sh '''
                /usr/bin/aws aws codeartifact login \
                 --tool maven \
                 --domain app-domain \
                 --domain-owner 654654304213 \
                 --repository sample_spring_boot_app_repo \
                 --region ap-south-1
                '''
           }
        }

        stage('Build Application') {
            steps {
                //Fetch dependencies + build JAR
                sh 'mvn clean package'
            }
        }

        stage('Upload Artifact to S3') {
            steps {
                //Store build output centrally
                sh '''
                aws s3 cp target/*.jar \
                s3://$S3_BUCKET/${JOB_NAME}/${BUILD_NUMBER}/
                '''
            }
        }
    }

    post {
        success {
            echo "Build successful and artifact uploaded to S3"
        }
        failure {
            echo "Build failed"
        }
    }
}

