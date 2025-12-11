pipeline {
    agent any

    //TODO1: jmeter 설치 폴더 위치 설정
    environment {
        JMETER_HOME_WIN = "C:/.../apache-jmeter-5.6.3"
        JMETER_HOME_UNIX = "/.../apache-jmeter-5.6.3"
    }

    stages {

        stage('Checkout JMX from GitHub') {
            steps {
                git branch: 'main',
                    //TODO2: JMeter 실행계획(.jmx)이 있는 Git Repo 주소
                    url: 'https://github.com/xxx/xxx.git'
            }
        }

        // Windows 에서도 동작하도록 bat분기 추가
        stage('Find JMX File') {
            steps {
                script {
                    def jmxFile = ""
        
                    if (isUnix()) {
                        jmxFile = sh(
                            script: "ls *.jmx 2>/dev/null | head -n 1",
                            returnStdout: true
                        ).trim()
                    } else {
                        jmxFile = bat(
                            script: 'for %%f in (*.jmx) do @echo %%f',
                            returnStdout: true
                        ).trim()
                    }
        
                    if (!jmxFile) {
                        error "❌ No .jmx file found in workspace!"
                    }
        
                    echo "✔ Found JMX file: ${jmxFile}"
                    env.JMX_FILE = jmxFile
                }
            }
        }

        stage('Run Performance Test') {
            steps {
                script {

                    def isWindows = !isUnix()
                    echo "Running on Windows? ${isWindows}"

                    def jmeterCmd = ""
                    if (isWindows) {
                        jmeterCmd = "\"${env.JMETER_HOME_WIN}/bin/jmeter.bat\""
                    } else {
                        jmeterCmd = "${env.JMETER_HOME_UNIX}/bin/jmeter"
                    }

                    // Cleanup old results
                    if (isWindows) {
                        bat "del /F /Q results.jtl 2>nul"
                        bat "rmdir /S /Q reports 2>nul"
                    } else {
                        sh "rm -f results.jtl"
                        sh "rm -rf reports"
                    }

                    // Execute JMeter with auto-detected JMX file
                    if (isWindows) {
                        bat """
                        ${jmeterCmd} -n ^
                          -t "${env.JMX_FILE}" ^
                          -l results.jtl ^
                          -e -o reports
                        """
                    } else {
                        sh """
                        ${jmeterCmd} -n \\
                          -t "${env.JMX_FILE}" \\
                          -l results.jtl \\
                          -e -o reports
                        """
                    }
                }
            }
        }

        //TODO3: Performance Plugin 설치 후 활성화
        /*
        stage('Publish Performance Report') {
            steps {
                // perfReport sources: 'results.jtl'
                
                perfReport(
                     sourceDataFiles: 'results.jtl',
                     parsers: [
                         [$class: 'JMeterParser', glob: 'results.jtl']
                     ]
                 )
    
            }
        }
        */


        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: 'results.jtl'
                archiveArtifacts artifacts: 'reports/**'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
