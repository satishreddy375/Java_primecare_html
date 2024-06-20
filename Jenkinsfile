pipeline {
    agent any
    tools {
        maven 'maven_3.9.4'
    }
    triggers{
        pollSCM('H/5 * * * *') // Polls the SCM (Git) every 5 minutes
    }
    stages {
        stage('git checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'Github_Credentials', poll: false, url: 'https://github.com/satishreddy375/Java_primecare_html.git'
            }
        }
        stage('maven package') {
            steps {
                sh "mvn clean package"
            }
        }
        stage('mvn sonar') {
            steps {
                sh "mvn clean sonar:sonar"
            }
        }
        stage('mvn nexus') {
            steps {
                sh "mvn clean deploy"
            }
        }
        stage('tomcat') {
            steps {
                script {
                    sshagent(['f1ace984-543b-411b-9e23-8d1bf715e4a2']) {
                        sh 'scp -o "StrictHostKeyChecking=no" /var/lib/jenkins/workspace/Declarative-pipeline/target/primecare-html.war/ ubuntu@13.201.22.128:/opt/apache-tomcat-9.0.88/webapps/'
                    }
                }
            }
        }
        stage('ansible host copying war file'){
            steps{
                sshagent(['Ansible_Credentials']) {
                    sh 'scp -o "StrictHostKeyChecking=no" /var/lib/jenkins/workspace/Declarative-pipeline/target/primecare-html.war/ ansadmin@3.108.42.7:/home/ansadmin'
                }
                
            }
        }
        stage('ansible playbook') {
            steps {
                script {
                    sshagent(['Ansible_Credentials']) {
                        sh 'ssh ansadmin@3.108.42.7 "ansible-playbook /home/ansadmin/docker_configs.yml"'
                    }
        }
    }
}
        stage('Pull Docker Image') {
            steps {
                script {
                    sshagent(['Docker_host']){
                        sh 'ssh dockeradmin@3.7.68.117 "docker pull satishreddy375/integration:latest"'
                    }
        }
    }
}
        stage('Docker Container') {
            steps {
                script {
                    sshagent(['Docker_host']){
                        sh 'ssh dockeradmin@3.7.68.117 "docker run -d --name docker_container -p 8080:8080 satishreddy375/integration:latest"'
                    }
        }
    }
}
        }
}
