pipeline {
  agent any
  stages {
    stage('Build') {
      parallel {
        stage('Server') {
          agent {
            docker {
              image 'maven:3.5-jdk-8-slim'
            }
            
          }
          steps {
            sh '''echo "Building the server code..."
mvn -version
mkdir -p target
touch "target/server.war"'''
            stash(name: 'server', includes: '**/*.war')
          }
        }
        stage('Client') {
          agent {
            docker {
              image 'node:6'
              args '-u 0:0'
            }
            
          }
          steps {
            sh '''echo "Building the client code..."
npm install --save react
mkdir -p dist
cat > dist/index.html <<EOF
hello!
EOF
touch "dist/client.js"'''
            stash(name: 'client', includes: '**/dist/*')
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Chrome') {
          agent {
            docker {
              image 'selenium/standalone-chrome'
            }
            
          }
          steps {
            sh 'echo \'mvn test -Dbrowser=chrome\''
          }
        }
        stage('Firefox') {
          agent {
            docker {
              image 'selenium/standalone-firefox'
            }
            
          }
          steps {
            sh 'echo \'mvn test -Dbrowser=firefox\''
          }
        }
      }
    }
    stage('QA') {
      agent {
        docker {
          image 'tomcat:8.0-jre8'
          args '-u 0:0 -p 11080:8080'
        }
        
      }
      steps {
        unstash 'server'
        unstash 'client'
        sh '''APP_DIR=/usr/local/tomcat/webapps
rm -rf $APP_DIR/ROOT
cp target/server.war $APP_DIR/server.war
mkdir -p $APP_DIR/ROOT
cp dist/* $APP_DIR/ROOT
/usr/local/tomcat/bin/startup.sh
'''
        input(message: 'Deploy?', ok: 'Go!!')
      }
    }
    stage('Deploy') {
      agent any
      steps {
        unstash 'client'
        unstash 'server'
        sh '''#Deploy teh exact artifacts to production
echo "Deploying client:"
ls -alFh dist
echo "Deploying server:"
ls -alFh target
'''
        echo 'Success!!!!'
      }
    }
  }
}