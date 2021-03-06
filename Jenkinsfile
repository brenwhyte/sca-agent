library "pipelineUtils"

def scaAgentZip

pipeline {
    agent {
        node {
            label 'docker'
        }
    }
    options {
        timestamps()
        disableConcurrentBuilds()
    }
    environment {
            VERSION = pipelineUtils.getSemanticVersion(0, 2)
    }
    stages{
        stage('Run E2E') {
            steps {
                script{
                    currentBuild.displayName = VERSION
                    e2eSecrets = pipelineUtils.getSCAAgentParams()
                    withEnv([
                        "JENKINS_NODE_COOKIE=dontkillMe",
                        "AUTHENTICATIONTOKENSOURCE=" + e2eSecrets.resource,
                        "SCOPE=" + e2eSecrets.scope,
                        "GRANT_TYPE=" + e2eSecrets.grantType,
                        "ACCESSCONTROLCLIENTID=" + e2eSecrets.clientId,
                        "SCATENANT=" + e2eSecrets.tenant,
                        "SCAUSERNAME=" + e2eSecrets.username,
                        "SCAPASSWORDSECRET=" + e2eSecrets.password,
                        "E2E_IMAGE_URL=" + e2eSecrets.e2eImageUrl]) {
                        sh(label: "Setup", script: "sh setup.sh")
                        sh("\$(aws ecr get-login --no-include-email --region eu-central-1)")
                        sh(label: "Run e2e", script: "sh dev/run-e2e.sh")
                    }
                }
            }
        }
        stage("Bundle") {
            steps {
                script{
                    scaAgentZip = "sca-agent.${VERSION}.zip"
                    sh(label: "Create bundle", script: "sh dev/bundle.sh ${scaAgentZip}")
                    archiveArtifacts artifacts: scaAgentZip
                }
            }
        }
    }
    post {
        success {
            script {
                if (env.BRANCH_NAME == "master") {
                    attachments = ["${WORKSPACE}/${scaAgentZip}"]
                    pipelineUtils.sendBuildStatusMail(null, attachments)
                }
            }
        }
    }
}
