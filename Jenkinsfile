deployProperties = [:]

pipeline {
    agent any
    environment {
        PROPERTIES_FILE_NAME = "deployment.properties"
    }
    parameters {
        booleanParam(name: 'RELEASE', defaultValue: false, description: 'Is this build for a release?')
        string(name: 'DEPLOY_BUILD_URL', defaultValue: 'http://localhost:8090/job/2287/7/', description: 'URL to jenkins deploy build to retrieve the `deployment.properties` file. If base parameters are defined, they will override the `deployment.properties` information')
    }
    stages {
        stage('Initialize deploy properties') {
            when {
                expression { return params.DEPLOY_BUILD_URL != '' }
            }
            steps {
                script {
                    readDeployProperties()
                    /* echo deployProperties.collect{ entry ->  "${entry.key}=${entry.value}" }.join("\n")
                    echo isRelease() */
                }
            }
        }
        stage('Approve promotion') {
            when {
                expression { return isRelease() && getDeployProperty('staging-repo.url') != '' }
            }
            steps {
                script {
                    def pipelineName = "Kogito Runtimes promote pipeline"
                    withCredentials([string(credentialsId: "KOGITO_CI_EMAIL_TO", variable: 'ZULIP_EMAIL')]) {
                        emailext body: "${pipelineName} #${env.BUILD_NUMBER} is ready for promotion.\n" +
                                 "The staging repository can be found at: ${getDeployProperty('staging-repo.url')}\n" +
                                 "Approve the promotion here: ${env.BUILD_URL}input",
                                 subject: "${pipelineName}",
                                 to: ZULIP_EMAIL
                    }
                    /* echo "${pipelineName} #${env.BUILD_NUMBER} is ready for promotion.\n" + 
                         "The staging repository can be found at: ${getDeployProperty('staging-repo.url')}\n" +
                         "Approve the promotion here: ${env.BUILD_URL}input"
                    echo "${pipelineName}"
                    input message: "Approve for promotion?" */
                }
            }
        }
    }
}

//////////////////////////////////////////////////////////////////////////////
// Deployment properties
//////////////////////////////////////////////////////////////////////////////

void readDeployProperties(){
    if (params.DEPLOY_BUILD_URL != null){
        withCredentials([usernamePassword(credentialsId: 'kmok-jenkins', usernameVariable: 'JENKINS_USER', passwordVariable: 'JENKINS_TOKEN')]) {
            sh "wget --auth-no-challenge --user=${JENKINS_USER} --password=${JENKINS_TOKEN} ${params.DEPLOY_BUILD_URL}artifact/${PROPERTIES_FILE_NAME}"
        }
        deployProperties = readProperties file: PROPERTIES_FILE_NAME
    }
}

boolean hasDeployProperty(String key){
    return deployProperties[key] != null
}

String getDeployProperty(String key){
    if(hasDeployProperty(key)){
        return deployProperties[key]
    }
    return ""
}

String getParamOrDeployProperty(String paramKey, String deployPropertyKey){
    if (params[paramKey] != ""){
        return params[paramKey]
    }
    return getDeployProperty(deployPropertyKey)
}

//////////////////////////////////////////////////////////////////////////////
// Getter / Setter
//////////////////////////////////////////////////////////////////////////////

boolean isRelease() {
    return params.RELEASE || getDeployProperty("release")
    /* return getParamOrDeployProperty("RELEASE", "release") */
}

String getProjectVersion() {
    return getParamOrDeployProperty("PROJECT_VERSION", "project.version")
}

String getGitTag() {
    return params.GIT_TAG != "" ? params.GIT_TAG : getProjectVersion()
}

String getBuildBranch() {
    return getParamOrDeployProperty("BUILD_BRANCH_NAME", "git.branch")
}
