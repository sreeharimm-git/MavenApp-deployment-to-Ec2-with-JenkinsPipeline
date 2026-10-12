pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sreeharimm-git/MavenApp-deployment-to-Ec2-with-JenkinsPipeline.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }



stage('Deploy to EC2') {
    steps {
        sshagent(['ssh-ubuntu']) {
            sh '''
                set -e

                HOST="ubuntu@13.61.149.199"
                JAR="target/demo-1.0.0.jar"

                echo "1. Checking SSH connection..."
                ssh -o BatchMode=yes \
                    -o StrictHostKeyChecking=no \
                    "$HOST" 'echo SSH connection successful'

                echo "2. Preparing application directory..."
                ssh "$HOST" \
                    'sudo mkdir -p /opt/app &&
                     sudo chown ubuntu:ubuntu /opt/app'

                echo "3. Copying JAR..."
                scp -o StrictHostKeyChecking=no \
                    "$JAR" "$HOST:/opt/app/"

                echo "4. Restarting Spring Boot..."
                ssh -o StrictHostKeyChecking=no "$HOST" \
                    'bash -s' <<'REMOTE'
set -e

cd /opt/app

# Stop the previous application if present.
if pgrep -f '[d]emo-1.0.0.jar' >/dev/null; then
    pkill -f '[d]emo-1.0.0.jar' || true
    sleep 2
fi

# Start the new version.
nohup java -jar /opt/app/demo-1.0.0.jar \
    > /opt/app/app.log 2>&1 < /dev/null &

sleep 5

if ! pgrep -f '[d]emo-1.0.0.jar' >/dev/null; then
    echo "Application failed to remain running."
    tail -n 50 /opt/app/app.log
    exit 1
fi

echo "Application process is running."
REMOTE

                echo "5. Deployment completed successfully."
            '''
        }
    }
}


    }

    post {
        success {
            echo "Deployment completed successfully."
        }
        failure {
            echo "Deployment failed. Check Jenkins or EC2 logs."
        }
    }
}
