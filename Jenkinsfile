@Library('jenkins-global-library') _
import java.net.URL
import linuxacademy.*

pipeline{
	agent none
	
	options {
		buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
	}
	
	environment {
		MAJOR_VERSION= 1
	}
	
	stages {
		
		stage("Say Hello"){
			agent any
			steps{
				def branch
    				def git = new linuxacademy.git.SayHello(this)
    				git.checkout()
			}
		}
		stage ('unit tests'){
			agent {
				label 'apache'
			}
			steps{
				sh 'ant -f test.xml -v'
				junit 'reports/result.xml'
			}
		}
		stage('build'){
			agent {
				label 'apache'
			}
			steps{
				sh 'ant -f build.xml -v'
			}
			
			post {
				success {
					archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
				}
			}
		}
		
		stage('deploy'){
			agent {
				label 'apache'
			}
			steps{
				sh "mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}"
				sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
			}
		}
		
		stage("Running on CentOS"){
			agent{
				label 'centos'
			}
			steps{
				sh "wget http://anirban6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
			}
		}
		
		stage("Test on Debian"){
			agent {
				docker "openjdk:8u121-jre"
			}
			steps{
				sh "wget http://anirban6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
			}
		}
		
		stage("promote to green"){
			agent {
				label 'apache'
			}
			when{
				branch "master"
			}
			steps{
				sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${BUILD_NUMBER}.jar"
			}
		}
		
	

	stage('Promote develop branch to master'){
	agent {
		label 'apache'
	}
	
	when {
		branch 'develop'
	}

	steps {
		echo "stahing any local changes"
		sh 'git stash'
		echo 'checking out develop branch'
		sh 'git checkout develop'
                sh 'git pull'
		echo 'Checking out master'
		sh 'git checkout master'
		echo 'Merging develop into master'
		sh 'git merge develop'
		echo 'pushing to origin master'
		sh 'git push origin master'
		echo 'Tagging the project'
		sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
		sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
	}

	post {
        	success {
                	emailext(
                        	subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!!",                      
                        	body: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!! Please look at the failure!!",
                        	to: "anirban25987@gmail.com"
                	)
        	}
	}


    }
	
   }
	post {
	        failure {
		     emailext(
			subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!!",			body: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!! Please look at the failure!!", 
			to: "anirban25987@gmail.com" 
		     )
	     }
}
}
