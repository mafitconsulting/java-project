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
     
     stage('Say Hello') {
        agent any

        steps {
           sayHello 'Awesome Fman'
        }
     }
     stage('Git Information') {
        agent any
     
        steps {
           echo "My branch name is: ${env.BRANCH_NAME}"
           script {
              def myLib = new mafitconsulting.git.gitStuff();
            
              echo "My commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
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
           sh "if [ ! -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ];then mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME};fi"
           sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/" }
     }
     stage("Running on CentOS") {
      agent {
         label 'CentOS'
       }
      steps {
          sh "echo http://fieldhousem6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
          sh "wget http://fieldhousem6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
          sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
     }
     stage("Test on Debian") {
         agent {
             docker 'openjdk:8u121-jre'
         }
         steps {
            sh "wget http://fieldhousem6.mylabserver.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
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
            sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
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
           echo "Stashing Any Local Changes"
           sh 'git stash'
           echo "Checking out development branch"
           sh 'git checkout development'
           echo "checking out master branch"
           sh 'git pull origin'
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
       post {
        success {
          emailext(
            subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Development Promoted to Master",
            body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Development Promoted to Master":</p>
            <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
            to: "Mark.Fieldhouse@mafitconsulting.co.uk"
          )
        }
       }
     }
  }
  post {
    failure {
      emailext(
        subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
        body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
        to: "Mark.Fieldhouse@mafitconsulting.co.uk"
      )
    }
  }
}
