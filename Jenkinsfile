pipeline {
    agent any

    environment {
        // 경로 설정을 아주 잘 하셨습니다. (역슬래시 2개 \\ 사용 필수)
        JMETER_HOME_WIN = "C:\\apache-jmeter-5.6.3"
        JMETER_HOME_UNIX = "/.../apache-jmeter-5.6.3"
    }

    stages {

        stage('Checkout JMX from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/dlxorud1256/jenkins_jmeter.git'
            }
        }

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
                        // [수정 1] PowerShell을 사용하여 파일명만 깔끔하게 가져옵니다.
                        // 기존 bat 방식은 지저분한 로그가 변수에 들어갈 위험이 큽니다.
                        jmxFile = powershell(
                            script: "(Get-ChildItem *.jmx | Select-Object -First 1).Name",
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
                        // 경로 구분자를 윈도우 표준(\)으로 변경
                        jmeterCmd = "\"${env.JMETER_HOME_WIN}\\bin\\jmeter.bat\""
                    } else {
                        jmeterCmd = "${env.JMETER_HOME_UNIX}/bin/jmeter"
                    }

                    // [수정 2] 파일이 있을 때만 삭제하도록 변경 (에러 방지)
                    // 그냥 del/rmdir을 쓰면 파일이 없을 때 빌드가 터집니다.
                    if (isWindows) {
                        bat "if exist results.jtl del /F /Q results.jtl"
                        bat "if exist reports rmdir /S /Q reports"
                    } else {
                        sh "rm -f results.jtl"
                        sh "rm -rf reports"
                    }

                    echo "▶ Starting JMeter Test..."

                    // Execute JMeter
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

        // [확인] Jenkins에 'Performance Plugin'이 설치되어 있어야 이 단계가 작동합니다.
        stage('Publish Performance Report') {
            steps {
                perfReport(
                     sourceDataFiles: 'results.jtl',
                     parsers: [
                         [$class: 'JMeterParser', glob: 'results.jtl']
                     ]
                 )
            }
        }

        stage('Archive Results') {
            steps {
                archiveArtifacts artifacts: 'results.jtl'
                // reports 폴더가 비어있어도 에러나지 않도록 옵션 추가
                archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
