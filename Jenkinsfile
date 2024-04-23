pipeline {

    agent {
     kubernetes {
       yamlFile 'kaniko-builder.yaml'
     }
    }
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        APP_NAME = "petclinic"
        RELEASE = "1.0.0"
        DOCKER_USER = "jmfrzier"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    
    stages{
        
        stage("Git Checkout"){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jmfrzier/spring-petclinic-docker.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }
       
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
/*        
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
*/        
        stage("Kaniko Build & Push"){
            environment {
                PATH = "/busybox:/kaniko:$PATH"
            }
            steps {
                container(name: 'kaniko', shell: '/busybox/sh') {
                    sh '''#!/busybox/sh
                    /kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=${IMAGE_NAME}:latest
                    '''          
            }
          }
        }
        
        stage("TRIVY"){
            steps{
                sh " curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b . v0.50.1" 
                sh " ./trivy image ${IMAGE_NAME}:latest"
            }
        }
/*        
        stage("Deploy To Tomcat"){
            steps{
                sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }
*/
    }
}
