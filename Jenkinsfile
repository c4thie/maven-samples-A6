pipeline {
    agent any
    tools { 
        maven 'DHT_MVN' 
        jdk 'DHT_SENSE' 
    }
    parameters {
        string(name: 'GOOD_COMMIT', defaultValue: '98ac319c0cff47b4d39a1a7b61b4e195cfa231e5', description: 'last known good commit')
        string(name: 'BAD_COMMIT', defaultValue: '198644632661c67b6c32f59e9047c11a70685e15', description: 'bad commit')
    }
    stages {
        stage('Check out') {
            steps {
                git(url: 'https://github.com/dhetong/maven-samples-A6.git', branch: 'master' )
            }
        }

        stage('Initial Verify') {
            steps {
                script {
                    try {
                        sh 'mvn verify'
                        echo "build successful so no need to bisect."
                    } catch (Exception e) {
                        echo "build failed! start automated bisect..."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Automated Bisect') {
            when {
                expression { currentBuild.result == 'UNSTABLE' }
            }
            steps {
                sh """
                    git bisect reset || true
                    git bisect start ${params.BAD_COMMIT} ${params.GOOD_COMMIT}
                    git bisect run mvn clean test
                    git bisect reset
                """
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}