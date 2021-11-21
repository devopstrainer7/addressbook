pipeline{

    agent any

    tools{
        jdk 'myjava'
        maven 'mymaven'
         dockerTool 'mydocker'
    }
    environment{
        NEW_VERSION='1.4.0'
    }
    stages{
        stage("COMPILE"){
           
            steps{
                script{
                     echo "Compiling the code"
                     git 'https://github.com/preethid/addressbook.git'
                     sh 'mvn compile'
                }
            }
                    }
        stage("UnitTest"){

           
              steps{

                script{
             echo "Run the unit test"
             sh 'mvn test'
        }
              }
              post{
                  always{
                      junit 'target/surefire-reports/*.xml'
                  }
              }
        }
        stage("Package"){

              steps{

                script{
              echo "Building the app"
              echo "building version ${NEW_VERSION}"
              sh 'mvn package'
        }
              }
        }

    stage("Builddockerimage"){      
        steps{
            script{
                echo "Containerising the app"
               withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh 'sudo docker build -t devopstrainer/java-mvn-privaterepos:$BUILD_NUMBER .'
                  sh 'sudo docker login -u $USERNAME -p $PASSWORD'
                 sh 'sudo docker push devopstrainer/java-mvn-privaterepos:$BUILD_NUMBER'
                }
            }
        }
    }
    stage("Deploydockercontainer"){
        steps{
            script{
              echo "Deploying the app"
             def dockerCmd = 'sudo docker run -itd -p 8001:80 devopstrainer/java-mvn-privaterepos:$BUILD_NUMBER'
                  sshagent(['deploy-server-ssh-key']) {
                      sh "ssh -o StrictHostKeyChecking=no ec2-user@13.235.115.181"
                      sh 'sudo systemctl start docker'
                      withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                           sh 'sudo docker login -u $USERNAME -p $PASSWORD'
                      sh "ssh -o StrictHostKeyChecking=no ec2-user@13.235.115.181 ${dockerCmd}"
                   }
                  }
                }
            }
        }
    }
        }
