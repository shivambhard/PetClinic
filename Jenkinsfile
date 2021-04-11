pipeline {
    
    agent {
        label 'master'
    }

    parameters {
        string defaultValue: '', description: '', name: 'env_name', trim: false
        choice choices: ['DEV', 'SIT', 'UAT', 'PROD'], description: '', name: 'environment'
    }
    
    tools {
        maven 'MAVEN3'
        jdk 'JAVA8'
    }

    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
        timestamps()
        disableConcurrentBuilds()
        parallelsAlwaysFailFast()
    }
    
    stages {
        
        stage('Use parameters'){
            steps {
                sh '''
                which mvn
                echo "print all env variables"
                env
                echo $env_name
                echo $environment
                '''
            }
        }
        
        stage ('Checkout Code'){
            steps {
                git credentialsId: 'github-credentials', url: 'https://github.com/gopishank/PetClinic.git'
            }
        }
        
        stage ('Unit Tests & SonarQube'){
            parallel {
                stage ('Unit Tests'){
                    steps {
                        sh "mvn test"
                    }
                }
                
                stage ('SonarQube') {
                    environment {
                        SCANNER_HOME = tool 'sonarqube-scanner'
                    }
                    steps {
                        withSonarQubeEnv( installationName: 'sonarqube-server') {
                            sh "$SCANNER_HOME/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                        }
                    }
                }
            }
        }  
        
        stage ('Package Code'){
            steps {
                sh "mvn package -Dmaven.test.skip=true"
            }
        }
        
        
        stage ('Upload Artifact'){
            steps {
                sh '''
                curl -u admin:admin123 POST "http://ec2-34-239-152-34.compute-1.amazonaws.com:8081/service/rest/v1/components?repository=petclinic" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "maven2.groupId=org.springframework.samples" -F "maven2.artifactId=petclinic" -F "maven2.version=${BUILD_ID}.0.0" -F "maven2.asset1=@${WORKSPACE}/target/petclinic.war" -F "maven2.asset1.extension=war"
                '''
            }
        }
        
        stage('Deploy Code'){
            steps {
                ansiblePlaybook installation: 'ANSIBLE29', playbook: '/opt/ansible-playbooks/deploy.yaml'
            }
        }
    }
}
