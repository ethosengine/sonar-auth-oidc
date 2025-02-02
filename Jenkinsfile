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
    image: maven:3.9.9-eclipse-temurin-21
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
      name: github-ee-bot-maven-settings
    '''
        }
    }
    
    parameters {
        booleanParam(name: 'IS_RELEASE', defaultValue: false, description: 'Check to perform a release')
        string(name: 'RELEASE_VERSION', defaultValue: '', description: 'Release version (e.g., 2.1.3.RELEASE)')
        string(name: 'NEXT_VERSION', defaultValue: '', description: 'Next development version (e.g., 2.1.4-SNAPSHOT)')
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
                           sh 'mvn clean verify sonar:sonar -Dmaven.test.failure.ignore=true'
                        }
                    }
                }
            }
        }
     
        stage('Release') {
            when {
                expression { return params.IS_RELEASE }
            }
            steps {
                container('maven-jdk21') {
                        sh """
    git config --global --add safe.directory \${WORKSPACE}

       git config --global user.email "mbd06b+ethosenginebot@gmail.com"
                    git config --global user.name "EthosengineBot"

             git checkout ${BRANCH_NAME}
                    git pull        
                            mvn -B release:prepare release:perform \
                                -DreleaseVersion=${params.RELEASE_VERSION} \
                                -DdevelopmentVersion=${params.NEXT_VERSION} \
                                -Darguments="-DskipTests" \
                                -DscmCommentPrefix="[maven-release-plugin][skip ci] "
                        """
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
