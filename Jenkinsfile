pipeline {
    agent any
    options {
        skipDefaultCheckout()
    }
    environment {
        // 본인의 JMeter 실제 설치 경로로 변경해 주세요 (터미널에서 'echo $JMETER_HOME' 또는 'which jmeter'로 확인)
        JMETER_HOME = '/opt/homebrew/opt/jmeter/libexec' 
        PATH = "${env.JMETER_HOME}/bin:${env.PATH}"
    }
    parameters {
        string(name: 'THREADS', defaultValue: '2', description: '가상 사용자 수 (5 이하)')
        string(name: 'LOOPS', defaultValue: '5', description: '반복 횟수 (10 이하)')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Smoke Test') {
            steps {
                sh '''
                    rm -rf reports html-report
                    mkdir -p reports
                    $JMETER_HOME/bin/jmeter -n -f -t perf/smoke.jmx -l reports/smoke.jtl -e -o html-report -Jthreads=$THREADS -Jloops=$LOOPS
                '''
            }
        }
        stage('Response Time Test') {
            steps {
                sh '$JMETER_HOME/bin/jmeter -n -f -t perf/duration.jmx -l reports/duration.jtl'
                perfReport sourceDataFiles: 'reports/duration.jtl', errorFailedThreshold: 5, errorUnstableThreshold: 1
                script {
                    if (currentBuild.currentResult == 'FAILURE') {
                        error('응답 시간 기준 미달: 에러율 5% 이상')
                    }
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'reports/**, html-report/**', allowEmptyArchive: true
        }
    }
}
