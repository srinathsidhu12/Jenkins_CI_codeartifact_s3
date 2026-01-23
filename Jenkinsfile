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
        stage('Configuring Maven for CodeArtifact....') {
  steps {
    sh '''#!/bin/bash
set -e

mkdir -p ~/.m2

export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token \
  --domain app-domain \
  --domain-owner 654654304213 \
  --region ap-south-1 \
  --query authorizationToken \
  --output text)

cat > ~/.m2/settings.xml <<EOF
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
  <servers>
    <server>
      <id>app-domain-sample_spring_boot_app_repo</id>
      <username>aws</username>
      <password>${CODEARTIFACT_AUTH_TOKEN}</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>app-domain-sample_spring_boot_app_repo</id>
      <url>https://app-domain-654654304213.d.codeartifact.ap-south-1.amazonaws.com/maven/sample_spring_boot_app_repo/</url>
      <mirrorOf>*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
EOF
'''
  }
} 
        stage('Build Application') {
            steps {
                //Fetch dependencies + build JAR
                sh 'mvn clean package -s ~/.m2/settings.xml'
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

