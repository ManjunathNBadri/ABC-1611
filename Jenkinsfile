pipeline {
    agent any

    environment {
        REPO_URL         = 'https://github.com/ManjunathNBadri/ABC-1611.git'
        REPO_ID          = 'adikarthikgupta '
        ARTIFACT_ID      = 'my-webapp'
        ARTIFACT_VERSION = '1.0-SNAPSHOT'
        WAR_PATH         = '/var/lib/jenkins/workspace/n1/my-webapp/my-webapp/target/my-webapp.war'
        REPO_URL_MAVEN   = ‘https://pkgs.dev.azure.com/adikarthikgupta/_packaging/adikarthikgupta/maven/v1’
        TOMCAT_DIR       = '/home/azureuser/tomcat/webapps'
        TOMCAT_SHUTDOWN  = '/home/azureuser/tomcat/bin/shutdown.sh'
        TOMCAT_STARTUP   = '/home/azureuser/tomcat/bin/startup.sh'
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "🔁 Cloning source code from GitHub..."
                git branch: 'master', url: "${REPO_URL}"
            }
        }

        stage('Build WAR with Maven') {
            steps {
                dir('my-webapp') {
                    echo "Building WAR file..."
                    sh 'mvn clean package -DskipTests'
                    sh 'ls -lh target/'
                }
            }
        }

        stage('Upload WAR to Azure Artifacts') {
            steps {
                withCredentials([string(credentialsId: 'adikarthikgupta', variable: 'AZURE_PAT')]) {
                    dir('my-webapp') {
                        echo "Creating temp settings.xml for Azure Artifacts..."
                        script {
                            writeFile file: 'settings-temp.xml', text: """
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>${REPO_ID}</id>
      <username> adikarthikgupta </username>
      <password>${AZURE_PAT}</password>
    </server>
  </servers>
</settings>
"""
                        }

                        echo "📦 Uploading WAR to Azure Artifacts..."
                        sh """
mvn deploy:deploy-file -s settings-temp.xml \
  -Dfile=target/my-webapp.war \
  -DgroupId=com.example \
  -DartifactId=${ARTIFACT_ID} \
  -Dversion=${ARTIFACT_VERSION} \
  -Dpackaging=war \
  -DrepositoryId=${REPO_ID} \
  -Durl=${REPO_URL_MAVEN}
"""
                    }
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Deploying WAR to Tomcat..."
                sh """
# Create Tomcat directory if it doesn't exist
if [ ! -d "${TOMCAT_DIR}" ]; then
  echo "Creating Tomcat webapps directory..."
  mkdir -p ${TOMCAT_DIR}
fi

# Copy WAR to Tomcat
echo "Copying WAR..."
cp ${WAR_PATH} ${TOMCAT_DIR}/

# Restart Tomcat
echo "Restarting Tomcat..."
${TOMCAT_SHUTDOWN} || true
sleep 5
${TOMCAT_STARTUP}
"""
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
        always {
            echo '🧹 Cleanup (if needed) complete.'
        }
    }
}

