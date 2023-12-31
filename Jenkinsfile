properties([
  parameters([
    [
      $class: 'ChoiceParameter',
      choiceType: 'PT_SINGLE_SELECT',
      name: 'Environment',
      script: [
        $class: 'ScriptlerScript',
        scriptlerScriptId:'Environments.groovy'
      ]
    ],
    [
      $class: 'CascadeChoiceParameter',
      choiceType: 'PT_SINGLE_SELECT',
      name: 'Host',
      referencedParameters: 'Environment',
      script: [
        $class: 'ScriptlerScript',
        scriptlerScriptId:'HostsInEnv.groovy',
        parameters: [
          [name:'Environment', value: '$Environment']
        ]
      ]
   ]
 ])
])

pipeline {
    agent any
    //triggers {
    //cron('*/2 * * * *')
    //}

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven-3"
        jdk "jdk-17"
    }

    stages {
        stage('Clean') {
            steps {
                cleanWs()   
            }
        }
        stage('Checkout') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', url: 'https://github.com/batkosz/szkolenie-ci-jenkins-example.git'

            }

        }
        stage('Build') {
            steps {
            echo "${params.Environment}"
            echo "${params.Host}"
            scmSkip(deleteBuild: true, skipPattern:'.*\\[ci skip\\].*')
            // Run Maven on a Unix agent.
            sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
    }

                post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/TEST-*.xml'
                    archiveArtifacts 'target/*.jar'
                    script {
                        slackSend(channel: "jenkins-BS", message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
                    }
                }
            }
}
