pipeline {
    agent any
    tools {
        maven 'maven'
        jdk 'JDK'
    }
    options {
        timestamps ()
        ansiColor('gnome-terminal')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    triggers {
        cron('0 6 * * 1-5')
    }
    
    parameters {
        string(name: 'TAG_NAME', defaultValue: "@employee", description: 'Scenario Tag to be run')
        choice(name: 'BRANCH_NAME', choices: ['master', 'sajnaNew'], description: 'Execution on branch') 
    }
    environment {
       JAVA_OPTS = "-Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400 -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.USE_BINARY_WRAPPER=true"
    }
    stages {
        stage('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }
        stage('Build') {
            steps {
                sh "mvn -f pom.xml -B -DskipTests clean package"
            }
            post {
                success {
//                     echo "Now Archiving the Artifacts....."
                    archiveArtifacts artifacts: '**/*.jar'
                }
            }
        }
        stage('Test') {
            steps {
                sh "mvn -f pom.xml test"
                sh "mvn clean verify -Dcucumber.filter.tags='$params.TAG_NAME' -DfailIfNoTests=false"
            }
//             post {
//                 always {
//                     junit 'Cucumber-Mvn-Project/target/surefire-reports/*.xml'
//                     html 'target/cucumber-report.html'
//                 }
//             }
        }
        stage('Cucumber Report') {
            steps {
                cucumber buildStatus: "UNSTABLE",
                    fileIncludePattern: "**/cucumber.json",
                    jsonReportDirectory: "target"
            }
        }
    }
}

