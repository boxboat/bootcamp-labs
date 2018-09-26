## Install Jenkins

1. Create New Instance   http://play.boxboat.net
2. Start Jenkins with docker access ```docker run --name jenkins --rm -d -p 8081:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/docker:/usr/bin/docker -u root jenkins/jenkins```
3. Get the admin password ```docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword```
4. Install reccomended plugins


## First Pipeline

In this section you will be creating your first pipeline.  It is a simple job that will echo "Hello World" to the output console.

1. Create a new pipeline in the GUI Home -> New Item -> Pipeline
2. Add Pipeline definition

Scroll to Definition:

```
node {
   echo 'Hello World'
   sh 'docker ps'
}
```
Click Save


3.  Build Now

Pipeline should have completed successfully.  View the logs by clicking on #1 on the left

## Multiple steps
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''
            }
        }
    }
}
```

## Docker Agents

```
pipeline {
    agent { docker { image 'python:3.5.1' } }
    stages {
        stage('build') {
            steps {
                sh 'python --version'
            }
        }
    }
}
```

## Environment Variables

```
pipeline {
    agent any

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                sh 'printenv'
            }
        }
    }
}
```

## Post Execution
```
pipeline {
    agent any

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                sh 'printenv'
            }
        }
    }
    post {
        always { echo 'always run' }
        success { echo 'if successful' }
        failure { echo 'if failed' }
        unstable { echo 'marked as unstable' }
        changed {
            echo 'state of the Pipeline has changed'
            echo 'ex, Pipeline was previously'
            echo 'failing but is now successful'
        }
    }
}
```

## Human Input
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'build step'
            }
        }
        stage('Test') {
            steps {
                sh 'docker run --name nginx-test -d -p 8888:80 nginx'
            }
        }
        stage('Approve') {
            steps {
                input "Approve for Production?"
            }
        }

        stage('Deploy to Prod') {
            steps {
                sh 'docker run --name nginx-prod -d -p 8889:80 nginx'
                sh 'sleep 120'
            }
        }
    }
    post {
        always { 
            sh 'docker stop nginx-test' 
            sh 'docker rm nginx-test'
            sh 'docker stop nginx-prod'
            sh 'docker rm nginx-prod'
            }
    }
}
```

## Git Plugin

```
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/boxboat/helloworld'
                sh 'ls -al'
            }
        }
    }
}
```

## Full Demo
```
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/boxboat/helloworld'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t hwwap:v0.0.3-test .'
                // docker push
            }
        }
        stage('Test') {
            steps {
                sh 'docker run --name hwwap-test -d -p 9000:80 hwwap:v0.0.3-test'

            }
        }
        stage('Approve') {
            steps {
                input "Approve for Production?"
            }
        }

        stage('Deploy to Prod') {
            steps {
                sh 'docker tag hwwap:v0.0.3-test hwwap:v0.0.3-prod'
                sh 'docker run --name hwwap-prod -d -p 9001:80 hwwap:v0.0.3-prod'
                sh 'sleep 120'
            }
        }
    }
    post {
        always { 
            sh 'docker stop hwwap-test' 
            sh 'docker rm hwwap-test'
            sh 'docker stop hwwap-prod'
            sh 'docker rm hwwap-prod'
            }
    }
}
```
