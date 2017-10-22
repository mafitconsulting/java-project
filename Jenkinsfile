pipeline {
   agent any

   options {
       buildDiscarder(logRotator(numToKeepStr: '2',artifactNumToKeepStr: '1'))
   }
   
   stages {
     stage('build') {
       steps {
       	  sh 'ant -f build.xml -v'
       }
     }
     stage('unit test') {
       steps {
           sh 'ant -f test.xml -v'
           junit 'reports/result.xml'
       }
     }
   }

   post {
     always {
        archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
     }
   }
}
   
