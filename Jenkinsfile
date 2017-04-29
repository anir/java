pipeline{
	agent none
	
	options {
		buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
	}
	
	stages {
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
				sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/"
			}
		}
		
		stage("Running on CentOS"){
			agent{
				label 'centos'
			}
			steps{
				sh "wget http://anirban6.mylabserver.com/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
			}
		}
		
		stage("Test on Debian"){
			agent {
				docker "openjdk:8u121-jre"
			}
			steps{
				sh "wget http://anirban6.mylabserver.com/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
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
				sh "cp /var/www/html/rectangles/all/rectangle_${BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${BUILD_NUMBER}.jar"
			}
		}
		
	}
	
	
}