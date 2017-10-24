pipeline {
   agent none

   environment {
     MAJOR_VERSION = 1
   }
   stages {
     stage('build') {
      agent {
         label 'apache'
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

     stage('unit test') {
       agent {
         label 'apache'
       }
       steps {
           sh 'ant -f test.xml -v'
           junit 'reports/result.xml'
       }
     }
     stage('deploy') {
       agent {
         label 'apache'
       }

       steps {
           sh "mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}"
           sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/" }
     }
     stage("Running on CentOS") {
      agent {
         label 'CentOS'
       }
      steps {
          sh "wget http://fieldhousem3.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
          sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
     }
     stage("Test on Debian") {
         agent {
             docker 'openjdk:8u121-jre'
         }
         steps {
            sh "wget http://fieldhousem3.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
            sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
        }
     }
     stage('Promote to green') {
       agent {
            label 'apache'
       }
       when {
            branch 'master'
       }
       steps {
            sh "cp /var/www/html/rectangles/all/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
       }
     }   
     stage('Promote Development branch to Master'){
       agent {
            label 'apache'
       }
       when {
            branch 'development'
       }
       steps {
           sh "Stashing any local changes" 
           sh 'git stash'
           echo "Checking out development branch"
           sh 'git checkout development'
           echo "checking out master branch"
           sh 'git checkout master'
           echo "Merging development into master branch"
           sh 'git merge development'
           echo "Pushing to Origin master"
           sh 'git push origin master' 
           echo 'Tagging the Release'
           sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
           echo 'Push to origin'
           sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
       }
     }
  }
}
