pipeline {
    agent any

    stages {
        stage('step1') {
            steps {
                echo 'Step1'
                sleep 4
            }
        }
        stage('step2') {
            steps {
                echo 'Step2'
                sleep 3
            }
        }
        stage('step3') {
            
          when { 
            branch "master"
            changeset "**/worker/**"
          }
            
            steps {
                echo 'Step3'
                sleep 5
            }
        }
    }
    post {
        always {
            echo "Pipeline is completed"
        }
        success { 
           discordSend description: 'Build successfull', footer: '', image: '', link: '', result: 'SUCCESS', scmWebUrl: '', thumbnail: '', title: '', webhookURL: 'https://discord.com/api/webhooks/1106442492049621013/G9LS3pX-cj4OE0fVo2UnxY4LwG8AwWwRPGyE-Fg9S80mI7dbvmFeflS8rSpOtlCz7Lyx'
        }
    }
}
