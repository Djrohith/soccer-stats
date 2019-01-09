#!groovy​

// FULL_BUILD -> true/false build parameter to define if we need to run the entire stack for lab purpose only
final FULL_BUILD = true
// HOST_PROVISION -> server to run ansible based on provision/inventory.ini
final HOST_PROVISION = params.HOST_PROVISION

final GIT_URL = 'https://github.com/Djrohith/soccer-stats.git'
final NEXUS_URL = '54.218.56.14:8081'

stage('Build') {
    node {
        git GIT_URL
        withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
            if(FULL_BUILD) {
                mvnHome = tool 'm3'
                sh "echo 'inside build'"
                
                 if (isUnix()) {
                     sh "mvn -B versions:set -DnewVersion=0.0.2-${BUILD_NUMBER}"
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
                
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Unit Tests') {   
        node {
            withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                sh "mvn -B clean test"
                stash name: "unit_tests", includes: "target/surefire-reports/**"
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Integration Tests') {
        node {
            withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                sh "mvn -B clean verify -Dsurefire.skip=true"
                stash name: 'it_tests', includes: 'target/failsafe-reports/**'
            }
        }
    }
}

if(FULL_BUILD) {
    stage('Static Analysis') {
        node {
            withEnv(["PATH+MAVEN=${tool 'm3'}/bin"]) {
                withSonarQubeEnv('sonar'){
                    unstash 'it_tests'
                    unstash 'unit_tests'
                    sh 'mvn sonar:sonar -DskipTests'
                }
            }
        }
    }
}
/**
if(FULL_BUILD) {
    stage('Approval') {
        timeout(time:3, unit:'DAYS') {
            input 'Do I have your approval for deployment?'
        }
    }
}**/


if(FULL_BUILD) {
    stage('Artifact Upload') {
        node {
           
            
            nexusArtifactUploader artifacts: [[groupId: 'soccer-versions', artifactId: 'soccer-stats', classifier: '', file: 'target/soccer-stats-0.0.2-${BUILD_NUMBER}.war', type: 'war']], credentialsId: '92a0b40b-83c4-4a1f-a901-a5859bbcb4a4', groupId: 'br.com.meetup.ansible', nexusUrl: '34.221.40.216:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'demoapp-rele', version: '0.0.2-${BUILD_NUMBER}'

             
        }
    }
}


stage('Deploy') {
    node {
        
        

       // http://34.221.40.216:8081/repository/demoapp-rele/br/com/meetup/ansible/soccer-stats/0.0.2-3/soccer-stats-0.0.2-3.war                           
        def artifactUrl = "http://${NEXUS_URL}/repository/demoapp-rele/br/com/meetup/ansible/soccer-stats/0.0.2-${BUILD_NUMBER}/soccer-stats-${BUILD_NUMBER}.war"

        withEnv(["ARTIFACT_URL=${artifactUrl}", "APP_NAME='soccer-demo'"]) {
            echo "The URL is ${env.ARTIFACT_URL} and the app name is ${env.APP_NAME}"

            // install galaxy roles
            sh "ansible-galaxy install -vvv -r provision/requirements.yml -p provision/roles/"    
            
            ansiblePlaybook become: true,
            colorized: true, credentialsId: 'ansible', disableHostKeyChecking: true, 
            installation: 'ansible', inventory: 'provision/inventory.ini', 
            playbook: 'provision/playbook.yml'

            /**ansiblePlaybook colorized: true, 
            credentialsId: 'ssh-jenkins',
            limit: "${HOST_PROVISION}",
            installation: 'ansible',
            inventory: 'provision/inventory.ini', 
            playbook: 'provision/playbook.yml', 
            sudo: true,
            sudoUser: 'jenkins' **/
        }
    } 
}
