pipeline {
    agent {
        label 'jenkinsslave1'
    }
    environment {
        FOODIES_GIT_PAT = credentials('foodiespat')
        TOMCAT_DOWNLOAD_URL = 'https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.91/bin/apache-tomcat-9.0.91.tar.gz'
        TOMCAT_BINARY_FILE = 'apache-tomcat-9.0.91.tar.gz'
        TOMCAT_HOME_DIR = '/u01/middleware/apache-tomcat-9.0.91'
    }
    tools {
        maven '3.8.7'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
        timestamps ()
        timeout(time: 1, unit: 'HOURS')
    }
    stages {
        stage('checkout') {
            steps {
                git url: "https://${FOODIES_GIT_PAT}@github.com/Raghu100994/foodies.git"
            }
        }
        stage('test') {
            steps {
                sh(script: "mvn --batch-mode -Dmaven.test.failure.ignore=true test")
            }
        }
        stage('package') {
            steps {
                sh(script: "mvn --batch-mode package -DskipTests")
            }
        }
        stage('configure') {
            input {
                message "Do you want to continue"
                ok "Yes"
            }
            steps {
                sh '''
                  sudo apt update -y
                  sudo apt install -y openjdk-11-jdk
                  COUNT=$(grep tomcat /etc/passwd | wc -l)
                  if [ $COUNT -eq 0 ]
                  then
                  sudo useradd -m -s /bin/bash tomcat
                  sudo mkdir -p /u01/middleware
                  sudo chown -R tomcat:tomcat /u01
                  fi
                  if [ ! -d "${TOMCAT_HOME_DIR}" ]
                  then
                  sudo su tomcat bash -c "wget ${TOMCAT_DOWNLOAD_URL} -P /u01/middleware/"
                  sudo su tomcat bash -c "tar -xvzf /u01/middleware/${TOMCAT_BINARY_FILE} -C /u01/middleware"
                  sudo cp src/main/config/tomcat.service.tmpl /etc/systemd/system/tomcat.service
                  JAVA_PATH=$(readlink -f $(which java))
                  JAVA_HOME=$(echo $JAVA_PATH | sed 's/bin.*//')
                  sudo sed -i 's|#JAVA_HOME_DIR#|'$JAVA_HOME'|g' /etc/systemd/system/tomcat.service
                  sudo sed -i 's|#TOMCAT_HOME_DIR#|'$TOMCAT_HOME_DIR'|g' /etc/systemd/system/tomcat.service
                  sudo sed -i 's|#TOMCAT_USER#|tomcat|g' /etc/systemd/system/tomcat.service
                  sudo sed -i 's|#TOMCAT_GROUP#|tomcat|g' /etc/systemd/system/tomcat.service
                  sudo systemctl daemon-reload
                  sudo systemctl enable tomcat
                  sudo systemctl start tomcat
                  fi
                  '''
            }
        }
        stage('deploy') {
            steps {
                sh '''
                sudo su tomcat bash -c "cp target/foodies.war $TOMCAT_HOME_DIR/webapps"
                sudo systemctl restart tomcat
                echo "***DEPLOYED***"
                '''
            }
        }
    }
}