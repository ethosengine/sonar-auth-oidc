pipeline {
    agent {
    	kubernetes {
            cloud 'kubernetes'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  nodeSelector:
    node-type: edge
  containers:
  - name: maven-jdk8
    image: maven:3.9.9-eclipse-temurin-8-alpine
    command:
    - cat
    tty: true
    volumeMounts:
    - name: maven-settings-volume
      mountPath: /usr/share/maven/ref/settings.xml
  - name: maven-jdk21
    image: maven:3.9.9-eclipse-temurin-21-alpine
    command:
    - cat
    tty: true
    volumeMounts:
    - name: maven-settings-volume
      mountPath: /usr/share/maven/ref/settings.xml
  volumes:
  - name: maven-settings-volume
    configMap:
      # Provide the name of the ConfigMap containing the files you want
      # to add to the container
      name: ghmbd06b-maven-settings
    '''
        }
    }
    
    stages {
    
        stage ('Chekout and Compile') {
                
            steps {
             	container('maven-jdk8'){
        	       script {
                        checkout scm
              
                        withMaven() {
                            sh 'mvn clean compile -fae'
                        }
                    }
                 }
            }
        }
        
        stage ('Unit Test') {
            steps {
                container('maven-jdk8'){
                    script {
                        withMaven() {
                            sh 'mvn test -fae'
                        }
                    }
                }
            }
        }
        
        stage ('Integration Test') {
            steps {
                container('maven-jdk8'){
                    script {
                        withMaven() {
                            sh 'mvn integration-test -fae'
                        }
                    }
                }
            }
        }
    
        stage ('QA Scan') {
            steps {
                container('maven-jdk21'){
                    script {
                        withSonarQubeEnv('ee-sonarqube') {
                           sh 'mvn clean verify sonar:sonar -Dmaven.test.failure.ignore=true -Dsonar.branch.name=${BRANCH_NAME}'
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            container('maven-jdk8') {
                junit(
                    allowEmptyResults: true,
                    testResults: '**/test-reports/*.xml'
                )
            }
        }
    }
}
