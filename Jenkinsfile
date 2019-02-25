#!groovy​

// FULL_BUILD -> true/false build parameter to define if we need to run the entire stack for lab purpose only
final FULL_BUILD = true
// HOST_PROVISION -> server to run ansible based on provision/inventory.ini
//final HOST_PROVISION = params.HOST_PROVISION
// limit: 'app_server' injecting by hardcoded
final HOST_PROVISION = '172.31.23.2'
 


final GIT_URL = 'https://github.com/Djrohith/soccer-stats.git'
final NEXUS_URL = '13.234.78.25:8081'

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
           
            
            nexusArtifactUploader artifacts: [[groupId: 'soccer-versions', 
                                               artifactId: 'soccer-stats', classifier: '',
                                               file: 'target/soccer-stats-0.0.2-${BUILD_NUMBER}.war', type: 'war']],
                credentialsId: '73509fe0-f24c-4642-ba5c-d55e1350b8aa', 
                groupId: 'br.com.meetup.ansible', nexusUrl: "${NEXUS_URL}", nexusVersion: 'nexus3', 
                protocol: 'http', repository: 'demoapp-rele', version: '0.0.2-${BUILD_NUMBER}'

             
        }
    }
}


stage('Deploy') {
    node {
        
        

       // http://54.70.187.156:8081/repository/demoapp-rele/br/com/meetup/ansible/soccer-stats/0.0.2-3/soccer-stats-0.0.2-3.war                           
        def artifactUrl = "http://${NEXUS_URL}/repository/demoapp-rele/br/com/meetup/ansible/soccer-stats/0.0.2-${BUILD_NUMBER}/soccer-stats-0.0.2-${BUILD_NUMBER}.war"

        withEnv(["ARTIFACT_URL=${artifactUrl}", "APP_NAME='soccer-stats'"]) {
            echo "The URL is ${env.ARTIFACT_URL} and the app name is ${env.APP_NAME}"

            // install galaxy roles
            //sh "ansible-galaxy install -vvv -r provision/requirements.yml -p provision/roles/"    
         
            sh "which ansible"
            sh "whoami"
         
          sh "ansible-galaxy install -vvv -r provision/requirements.yml -p provision/roles/" 
         
            sh "ansible -m ping app_server"
           // sh "ansible-playbook  provision/playbook.yml --extra-vars \" ARTIFACT_URL=${artifactUrl} APP_NAME='soccer-stats' \" "       
         
       // sh "ansible-playbook provision/playbook.yml --extra-vars \"  APP_NAME=soccer-stats ARTIFACT_URL=http://54.70.187.156:8081/repository/demoapp-rele/br/com/meetup/ansible/soccer-stats/0.0.41-/soccer-stats-0.0.2-41.war\" 
     
         
           // ansiblePlaybook colorized: true, 
            //credentialsId: 'playbook',
            //limit: "${HOST_PROVISION}",
            //installation: 'ansible',
            //inventory: 'provision/inventory.ini', 
            //playbook: 'provision/playbook.yml', 
            //sudo: true,
            //sudoUser: 'root' //
                  
       
            //ansiblePlaybook colorized: true, 
            //credentialsId: 'playbook',
            //limit: "${HOST_PROVISION}",
            //installation: 'ansible',
            //inventory: 'provision/inventory.ini', 
            //playbook: 'provision/playbook.yml'
            //sudo: true,
            //sudoUser: 'jenkins'
               
     
                //ansiblePlaybook become: true, colorized: true, 
                //credentialsId: 'ansible', disableHostKeyChecking: true,              
                //extras: 'ARTIFACT_URL="${artifactUrl}" APP_NAME=soccer-demo',
                //inventory: 'provision/inventory.ini',
                //playbook: 'provision/playbook.yml',
                //sudoUser: 'ec2-user' 
                
            
            ansiblePlaybook colorized: true, 
            credentialsId: 'ssh-jenkins',
           // limit: "${HOST_PROVISION}",
            installation: 'ansible',
            inventory: 'provision/inventory.ini', 
            playbook: 'provision/playbook.yml',
            //extra-vars: APP_NAME=soccer-stats ARTIFACT_URL=http://54.70.187.156:8081/repository/demoapp-rele/br/com/meetup/ansible/soccer-stats/0.0.41-/soccer-stats-0.0.2-41.war\" "
            //sudo: true,
             sudoUser: 'jenkins',
             extraVars: [
              ARTIFACT_URL : env.ARTIFACT_URL 
              ]
           
             
           
        }
    } 
}
