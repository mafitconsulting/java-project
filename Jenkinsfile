pipeline {
   agent none

   options {
       buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeepStr: '1'))
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
           sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all"
       }
     }
     stage("Running on CentOS") {
      agent {
         label 'CentOS'
       }
      steps {
          sh "wget http://fieldhousem3.mylabserver.com/retangles/all/rectangle_${env.BUILD_NUMBER}.jar"
          sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
      }
     }
  }
}
   
