pipeline
{
 agent {
    label 'helm'
 }
environment
{
    SONAR_HTTP_URL='http://3.83.218.79:9000'
    JFROG_REPO_URL="devops1947.jfrog.io"
    JFROG_REPO_NAME="docker-local"

    APP_NAME="netflix"
    APP_VER="1.0.0"

    IMAGE_NAME="${JFROG_REPO_URL}" + "/" + "${JFROG_REPO_NAME}" + "/" + "${APP_NAME}"
    IMAGE_TAG="${APP_VER}" + "-" + "${BUILD_NUMBER}"

    TMDB_API_KEY = credentials('tmdb_api') //Env accepts secret text
    JFROG_DOCKER_TOKEN = credentials('jfrog_docker_token')
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
        post
        {
            failure
            {
                error "sonar code quality check failure"
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
            sh 'npm install'
        }
    }
    stage('Docker Build')
    {
        steps
        {
            script
            {
                sh '''
                docker image build --build-arg TMDB_V3_API_KEY="${TMDB_API_KEY}" . -t "${APP_NAME}":latest
                '''
            }
        }
        post
        {
            success
            {
                sh '''
                echo "\n=================================================================================\n"
                echo "\n==================================TRIVY-SCANE====================================\n"
                echo "\n=================================================================================\n"
                trivy image "${APP_NAME}":latest
                '''
            }
        }
    }
    stage('docker image push to jfrog')
    {
        steps
        {
                //withCredentials([string(credentialsId: 'jfrog_docker_token', variable: 'JFROG_DOCKER_TOKEN')]) 
                //{
                    script
                    {
                        sh '''
                            docker login -u duraipandian.indrajith@nttdata.com -p "${JFROG_DOCKER_TOKEN}" devops1947.jfrog.io
                            docker image tag "${APP_NAME}":latest "${IMAGE_NAME}":"${IMAGE_TAG}"
                            
                        '''
                        // Push image to JFROG artifactory
                        def pushStatus = sh (script: "docker push ${IMAGE_NAME}:${IMAGE_TAG}", returnStatus: true)
                        
                        //Docker push failure
                        if(pushStatus != 0)
                        {
                            echo "Docker image push failed with the status: ${pushStatus}"
                        }
                        //docker push "${IMAGE_NAME}":"${IMAGE_TAG}"

                    }
                //}

        }
        post
        {
            success
            {
                sh '''
                echo "\n==========================================================================================\n"
                echo "\n==================================IMAGE-PUSH-COMPLETED====================================\n"
                echo "\n==========================================================================================\n"
                '''
            }
        }
    }
 }
}