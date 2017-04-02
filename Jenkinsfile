pipeline {
    agent none
    environment {
	MAJOR_VERSION=1	
    }	
    options {
	    buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeepStr:'1'))
    }
    stages {
	 stage('Unit Test') {
	        agent {
                label 'master'
            }
            steps {
                sh 'ant -f test.xml -v'
		junit 'reports/result.xml'
            }
        }    
        stage('build') {
            agent {
                label 'master'
            }
            steps {
                sh 'ant -f build.xml -v'
            }
            post {
        success {
            archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
    }
        }
	stage('deploy') {
	        agent {
                label 'master'
            }
            steps {
		    sh "mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}"
		    sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
            }
        }
    stage('Running on CentOS') {
	        agent {
                label 'Centos'
            }
            steps {
            sh "wget http://54.145.133.138/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
		    sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
            }
        }    
    stage('Running on Debian') {
	        agent {
                docker 'openjdk:8u121-jre'
            }
            steps {
            sh "wget http://54.145.133.138/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.BUILD_NUMBER}.jar"
		    sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
            }
        }  	    
    stage('Promote to Green') {
	    agent {
                label 'master'
            }
	    when {
	    	branch 'master'
	    }
	    steps {
		    sh "cp /var/www/html/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/"
            }
    }
    stage('Merging development branch to master'){
	     agent {
                label 'master'
            }
	    when {
	    	branch 'development'
	    }
	    steps {
		  echo "Stashing any local change"
		  sh "git stash"  
		  echo "Checking out development branch"
		  sh "git checkout development"
		  echo "Checking out development branch"  
		  sh "git checkout master"  
		  echo "Merging development branch with master"  
		  sh "git merge development"    
		  sh "git push origin master"  
		  sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}" 
		  sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"  
	    }
	    
	    }	    
    }
     
}
