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
        stage('checkout') {
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
                        // Apply Java version fix for initial build
                        sh '''
                            find . -name 
"pom.xml" -exec sed -i.bak \'s/<source>1.6<\\/source>/<source>1.8<\\/source>/g\' {} + || true
                            find . -name "pom.xml" -exec sed -i.bak \'s/<target>1.6<\\/target>/<target>1.8<\\/target>/g\' {} + || true
                        '''
                        sh 'mvn clean test'
                        echo "build and test passed"
                        currentBuild.result = 'SUCCESS'
                    } catch (Exception e) {
                        echo "initial build and test failed, needbisect"
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
# Use Python to update Java version - more robust than sed across platforms
python3 -c "
import os
for root, dirs, files in os.walk('.'):
    for file in files:
        if file == 'pom.xml':
            path = os.path.join(root, file)
            with open(path, 'r') as f:
                content = f.read()
            new_content = content.replace('<source>1.6</source>', '<source>1.8</source>')
            new_content = new_content.replace('<target>1.6</target>', '<target>1.8</target>')
            if content != new_content:
                with open(path, 'w') as f:
                    f.write(new_content)
                print(f'Updated {path}')
"

mvn clean test
exit $?
EOF
                    '''
                    sh 'chmod +x bisect_test.sh'

                    // Run the bisect process
                    sh "git bisect reset || true"
                    sh "git bisect start ${params.BAD_COMMIT} ${params.GOOD_COMMIT}"
                    sh "git bisect run ./bisect_test.sh"
                    
                    // Final cleanup
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
