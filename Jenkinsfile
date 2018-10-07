pipeline {
	options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    post { always { deleteDir() } }
    agent any
    stages {
        stage('Deploy') {
            when { expression { params.DEPLOY_JOB_NAME == null } }
            steps {
                echo "OK"
            }
        }
        stage('Docker') {
            when { expression { params.DEPLOY_JOB_NAME != null } }
            steps {                
                script {
                    def JOB_TAG_SECTIONS = [params.DEPLOY_JOB_NAME, "master"]
                    def NAME = JOB_TAG_SECTIONS[0]
                    def BRANCH = JOB_TAG_SECTIONS[1].replace("/", ".")
                    def REGISTRY_URL = "${params.REPOSITORY}/${NAME}:${BRANCH}"
                    def ENVIRONMENT = params.ENVIRONMENT
                    def COMPOSE_YML = "docker-compose.${ENVIRONMENT}.yml"
                    

                    echo """
==> Configuration
--> Names
Job name: ${params.DEPLOY_JOB_NAME}
Project name: ${NAME}
Branch: ${BRANCH}
Repository URL: ${REGISTRY_URL}
Environment: ${ENVIRONMENT}

--> Directories:
Working directory: ${params.SOURCE_DIRECTORY}
                    
                    """
                    
                    if ("${BRANCH}" =~ /^(master|develop|feature*)/) {
                    
                        if (fileExists("${params.SOURCE_DIRECTORY}/laradock/${COMPOSE_YML}")) {
                            echo "--> Found ${COMPOSE_YML};"

                            echo "--> Customizing domain names for branch ${BRANCH}"                                                    
                            sh "sed -i -e \"s/Host:/Host:${BRANCH}./g\" ${params.SOURCE_DIRECTORY}/${COMPOSE_YML}"
                            sh "sed -i -e \"s/:master/:${BRANCH}/g\" ${params.SOURCE_DIRECTORY}/${COMPOSE_YML}"

                            echo "--> Deploying"
                            sh "docker stack deploy -c ${params.SOURCE_DIRECTORY}/laradock/${COMPOSE_YML} --prune ${NAME}-${BRANCH}"
                        } else {
                            echo "--> Did not find ${COMPOSE_YML};"
                            error("--> Looks like this project is not configured for running as ${ENVIRONMENT}")
                        }
                    } else {
                        echo "--> ${BRANCH} is an unsupported branch name for automation."
                    }
                }
            }
        }
    }
}

