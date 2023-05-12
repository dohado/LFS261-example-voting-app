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
    }
}
