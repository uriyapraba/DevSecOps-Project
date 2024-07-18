pipeline
{
 agent {
    label 'helm'
 }
environment
{
    SONAR_HTTP_URL='http://44.211.131.56:9000'
}
 stages 
 {
    stage('CleanWS') {
        steps {
            cleanWs()
        }
    }
    stage('SCM checkout') 
    {
        steps {
            script {
            checkout scmGit(branches: [[name: 'origin/dev']], 
            extensions: [], userRemoteConfigs: [[credentialsId: 'git_token', url: 'https://github.com/uriyapraba/DevSecOps-Project.git']])
            }
        }
    }
    stage('sonar codequality check') {
        steps
        {
            withCredentials([string(credentialsId: 'sonar_netflix_report_token', variable: 'SONAR_AUTH_TOKEN')]) 
            {
                script
                {
                    def scannerHome = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
                    withSonarQubeEnv('sonarqube') 
                    {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=netflix_new \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HTTP_URL} \
                            -Dsonar.login=${SONAR_AUTH_TOKEN}
                        """
                    }
                }
            }
        }
    }
    /*stage('sonar quality gate')
    {
        steps
        {
                script
                {
                def qg = waitForQualityGate()
                if(qg.status != "OK")
                    {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            
        }
    }*/
    stage('Install dependancy')
    {
        steps
        {
            sh 'npm -v'
        }
    }
 }
}