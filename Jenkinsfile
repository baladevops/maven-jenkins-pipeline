node {
  checkout scm
  env.PATH = "${tool 'Maven3'}/bin:${env.PATH}"
  stage('Package') {
    dir('webapp') {
      sh 'mvn clean package -DskipTests'
    }
  }

  stage('Create Docker Image') {
    dir('webapp') {
      docker.build("webapp/docker-jenkins-pipeline:${env.BUILD_NUMBER}")
    }
  }
  
  stage('JaCoCo') {
            echo 'Code Coverage'
            jacoco()
        }
  stage('Sonar') {
            echo 'Sonar Scanner'
            //def scannerHome = tool 'SonarQube Scanner 3.0'
	    withSonarQubeEnv('SonarQube Server') {
		    sh '/root/sonar-scanner-3.2.0.1227-linux/bin/sonar-scanner'
	    }
        }

  stage ('Run Application') {
   // try {
      // Start database container here
      // sh 'docker run -d --name db -p 8091-8093:8091-8093 -p 11210:11210 arungupta/oreilly-couchbase:latest'

      // Run application using Docker image
      // sh "DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`"
      // sh "docker run -e DB_URI=$DB webapp/docker-jenkins-pipeline:${env.BUILD_NUMBER}" 
      // sh "docker run -e DB_URI=$(docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db) webapp/docker-jenkins-pipeline:${env.BUILD_NUMBER}"
         sh '''#!/bin/sh                 
                 DB=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`
                 docker run -e DB_URI=${DB} webapp/docker-jenkins-pipeline:${BUILD_NUMBER}
         '''
      // Run tests using Maven
      //dir ('webapp') {
      //  sh 'mvn exec:java -DskipTests'
      //}
    //} catch (error) {
    //} finally {
      // Stop and remove database container here
      //sh 'docker-compose stop db'
      //sh 'docker-compose rm db'
    //}
  }

  stage('Run Tests') {
    try {
      dir('webapp') {
	sh '''#!/bin/sh                 
                 export DB_URI=`docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' db`
                 mvn test
         '''
        //sh "mvn test"
       // docker.build("webapp/docker-jenkins-pipeline:${env.BUILD_NUMBER}").push()
      }
    } catch (error) {

    } finally {
      junit '**/target/surefire-reports/*.xml'
    }
  }
}
