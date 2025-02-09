pipeline {
    agent none
    stages{
      
      stage("worker_build"){
        agent {
          docker {
            image 'maven:3.9.2-eclipse-temurin-11'
            args '-v $HOME/.m2:/root/.m2'
          }   
        }
        when {
                changeset "**/worker/**"
        }
        steps{
          echo 'Compiling worker app inside maven container'
          dir('worker'){
            sh 'mvn compile'
          }
        }
      }
        
      stage("worker_test"){
        agent {
          docker {
            image 'maven:3.9.2-eclipse-temurin-11'
            args '-v $HOME/.m2:/root/.m2'
          }   
        }
        when {
          changeset "**/worker/**"
        }
        steps{
          echo 'Running Unit Tests on worker app'
          dir('worker'){
            sh 'mvn clean test'
          }
        }
      }
      
      stage("worker_package"){
        agent {
          docker {
            image 'maven:3.9.2-eclipse-temurin-11'
            args '-v $HOME/.m2:/root/.m2'
          }   
        }
        when {
          branch "master"
          changeset "**/worker/**"
        }
        steps{
          echo 'Packaging worker app'
          dir('worker'){
            sh 'mvn package -DskipTests'
            archiveArtifacts artifacts: "**/target/*.jar", fingerprint: true
          }
        }
      }
        
      stage('worker_docker-package'){
        agent any
        when {
          branch "master"
          changeset "**/worker/**"
        }
        steps{
          echo 'Packaging worker app with docker'
          script{
            docker.withRegistry('https://index.docker.io/v1/','dockerlogin'){
              def workerImage = docker.build("dehado/worker:v${env.BUILD_NUMBER}", "./worker")
              workerImage.push()
              workerImage.push('latest')
            }
          }
        }
      }
      
      stage("result_build"){
        agent {
          docker {
            image 'node:8.16.0-alpine'
          }
        }   
        when {
          changeset "**/result/**"
        }
        steps{
          echo 'Building worker app in container'
          dir('result'){
            sh 'npm install'
          }
        }
      }

      stage("result_test"){
        agent {
          docker {
            image 'node:8.16.0-alpine'
          }
        }  
        when {
          changeset "**/result/**"
        }
        steps{
          echo 'Running Unit Tests on worker app'
          dir('result'){
            sh 'npm install; npm test'
          }
        }
      }

      stage('result_docker-package'){
        agent any
        when {
          branch "master"
          changeset "**/result/**"
        }
        steps{
          echo 'Packaging result app with docker'
          script{
            docker.withRegistry('https://index.docker.io/v1/','dockerlogin'){
              def resultImage = docker.build("dehado/result:v${env.BUILD_NUMBER}", "./result")
              resultImage.push()
              resultImage.push('latest')
            }
          }
        }
      }

      stage('vote-build') {
        agent {
          docker {
            image 'python:2.7.16-slim'
            args '--user root'
          }
        }
        when {
          changeset '**/vote/**'
        }
        steps {
          echo 'Compiling vote app.'
          dir(path: 'vote') {
            sh 'pip install -r requirements.txt'
          }
        }
      }

      stage('vote-test') {
        agent {
          docker {
            image 'python:2.7.16-slim'
            args '--user root'
          }
        }
        when {
          changeset '**/vote/**'
        }
        steps {
          echo 'Running Unit Tests on vote app.'
          dir(path: 'vote') {
            sh 'pip install -r requirements.txt'
            sh 'nosetests -v'
          }
        }
      }

      stage('vote integration'){ 
        agent any 
        when{ 
          changeset "**/vote/**" 
          branch 'master' 
        } 
        steps{ 
          echo 'Running Integration Tests on vote app' 
          dir('vote'){ 
            sh 'sh integration_test.sh' 
          } 
        } 
      } 

      stage('vote-docker-package') {
        agent any
        when {
          branch "master"
          changeset "**/vote/**"
        }
        steps {
          echo 'Packaging vote app with docker'
          script {
            docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
              def voteImage = docker.build("dehado/vote:${env.BUILD_NUMBER}", "./vote")
              voteImage.push()
              voteImage.push('latest')
            }
          }
        }
      }

      stage('Sonarqube') {
        agent any
        when{
          branch 'master'
        }
        // tools {
       // jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
       // }

        environment{
          sonarpath = tool 'SonarScanner'
        }

        steps {
          echo 'Running Sonarqube Analysis..'
          withSonarQubeEnv('sonar-instavote') {
            sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
          }
        }
      }


      stage("Quality Gate") {
        steps {
          timeout(time: 1, unit: 'HOURS') {
                // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                // true = set pipeline to UNSTABLE, false = don't
            waitForQualityGate abortPipeline: true
          }
        }
      }

      // stage('deploy to dev') {
      //   agent any
      //   when {
      //     branch 'master'
      //   }
      //   steps {
      //     echo 'Deploy instavote app with docker compose'
      //     sh 'docker-compose up -d'
      //   }
      // }

      stage('Trigger deployment') {
        agent any
        environment{
          def GIT_COMMIT = "${env.GIT_COMMIT}"
        }
        steps{
          echo "${GIT_COMMIT}"
          echo "triggering deployment"
          // passing variables to job deployment run by vote-deploy repository Jenkinsfile
          build job: 'deployment', parameters: [string(name: 'DOCKERTAG', value: GIT_COMMIT)]
        }    
      }     
    
    }

    post{
        always{
            echo 'Building monopipe for microservice vote app is completed..'
        }
    }
}
