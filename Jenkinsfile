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
               sh 'which aws && aws --version && echo $PATH'
           }
        }         
        stage('Fetch CodeArtifact Token') {
           steps {
             script {
               env.CODEARTIFACT_AUTH_TOKEN = sh(
                script: """
                aws codeartifact get-authorization-token \
                  --domain ${CODEARTIFACT_DOMAIN} \
                  --domain-owner ${ACCOUNT_ID} \
                  --region ${AWS_REGION} \
                  --query authorizationToken \
                  --output text
                """,
                returnStdout: true
              ).trim()
            }
            echo "Token fetched successfully"
          }
        }
        stage('Create Maven settings.xml') {
            steps {
                sh '''
                mkdir -p ~/.m2
                cat > ~/.m2/settings.xml <<EOF
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      https://maven.apache.org/xsd/settings-1.0.0.xsd">

 <servers>
  <server>
    <id>app-domain-sample_spring_boot_app_repo</id>
    <username>aws</username>
    <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
  </server>
 </servers>
 <mirrors>
  <mirror>
    <id>app-domain-sample_spring_boot_app_repo</id>
    <name>app-domain-sample_spring_boot_app_repo</name>
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

