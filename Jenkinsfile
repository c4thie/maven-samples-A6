pipeline {
    agent any
    tools {
        maven 'DHT_MVN'
        jdk 'DHT_SENSE'
    }
    parameters {
        string(name: 'GOOD_COMMIT', defaultValue: '98ac319c0cff47b4d39a1a7b61b4e195cfa231e5', description: 'last known good commit hash')
        string(name: 'BAD_COMMIT', defaultValue: '198644632661c67b6c32f59e9047c11a70685e15', description: 'bad commit hash')
    }
    stages {
        stage('Checkout Source') {
            steps {
                script {
                    sh 'git clean -fdx || true'
                    sh 'git reset --hard || true'
                    git(url: 'https://github.com/dhetong/maven-samples-A6.git', branch: 'master' )
                }
            }
        }

        stage('initial fix for source and target version') {
            steps {
                script {
                    try {
                        sh '''
                            find . -name "pom.xml" -exec sed -i 's/<source>1.6<\/source>/<source>1.8<\/source>/g' {} + || true
                            find . -name "pom.xml" -exec sed -i 's/<target>1.6<\/target>/<target>1.8<\/target>/g' {} + || true
                        '''
                        sh 'mvn clean test'
                        echo "initial build and test passed."
                        currentBuild.result = 'SUCCESS'
                    } catch (Exception e) {
                        echo "initial build and test failed, need bisect."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('git bisect') {
            when {
                expression { currentBuild.result == 'UNSTABLE' }
            }
            steps {
                script {
                    sh '''
                        cat > bisect_test.sh <<'EOF'
#!/bin/bash
# Fix the Java version issue in pom.xml for the current commit
find . -name "pom.xml" -exec sed -i 's/<source>1.6<\/source>/<source>1.8<\/source>/g' {} + || true
find . -name "pom.xml" -exec sed -i 's/<target>1.6<\/target>/<target>1.8<\/target>/g' {} + || true

# Run the actual tests
mvn clean test

# The exit code of mvn clean test will be returned to git bisect
exit $?
EOF
                    '''
                    sh 'chmod +x bisect_test.sh'
                    sh "git bisect reset || true"
                    sh "git bisect start ${params.BAD_COMMIT} ${params.GOOD_COMMIT}"
                    sh "./bisect_test.sh"
                    sh "git bisect reset"
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
