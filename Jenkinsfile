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
        stage('Get CodeArtifact Token') {
            steps {
                sh'''
                export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token \
                 --domain app-domain \
                 --domain-owner ACCOUNT_ID \
                 --query authorizationToken \
                 --output text)
               '''
            }
        }

        stage('Build Application') {
            steps {
                //Maven must know which settings file to use(contains how to authenticate with codeartifact.
                //Fetch dependencies + build JAR
                sh '''
                 mvn clean package' /
                -s /var/lib/jenkins/.m2/settings.xml
                '''
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

