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
               aws --versio
               '''
           }
        }         
        stage('Authenticate to CodeArtifact') {
            steps {
                //Fetches auth token
                sh """
                /usr/local/aws-cli/v2/current/bin/aws codeartifact login \
                 --tool maven \
                 --domain $CODEARTIFACT_DOMAIN \
                 --domain-owner $ACCOUNT_ID \
                 --repository $CODEARTIFACT_REPO \
                 --region $AWS_REGION
                """
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

