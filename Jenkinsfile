def ENVMAP = [
    "devbuild":       ["AccountId":"992732154600", "AssumeRoleName": "devops-dev-opsjenkinsslave-iam-role",  "VersionUpdateEnvironment": "dev"],
    "releasebuild":   ["AccountId":"664912803900", "AssumeRoleName": "devops-sit-opsjenkinsslave-iam-role",  "VersionUpdateEnvironment": "sit"],
    "hotfixbuild":    ["AccountId":"664912803900", "AssumeRoleName": "devops-uat-opsjenkinsslave-iam-role",  "VersionUpdateEnvironment": "uat"]
]
 
pipeline {
    agent any
    options {
        ansiColor("xterm")
        buildDiscarder(logRotator(numToKeepStr:"5"))
        disableConcurrentBuilds()
        timeout(time: 15, unit: "MINUTES")
        timestamps()
    }
    parameters {
        choice(
            choices: ["devbuild"],
            description: "Build from develop branch, start a release or finish from release branch",
            name: "REQUESTEDACTION")
    }
    environment {
        PROJECT_GIT_GROUP = "praveenkumardevops07"
        VERSION_REGISTRY_REPO = "versionregistry"
        VERSION_REGISTRY_BRANCH = "master"
        APPLICATIONREPO = "login-service"
        APPLICATIONREPO_BRANCH = "master"
        REQUESTEDACTION = "${params.REQUESTEDACTION}"
		GIT_CRDS = "GIT"
		GIT_VR_URI = "https://github.com/${PROJECT_GIT_GROUP}/${VERSION_REGISTRY_REPO}"
		GIT_TP_URI = "https://github.com/${PROJECT_GIT_GROUP}/${APPLICATIONREPO}"
      }  
      stages {
        stage ("Checkout SCM") {
              steps {
                  script {
                      ansiColor("xterm") {					
					  dir('VERSION_REGISTRY_REPO'){
					  checkout (
						scm: [
							$class: 'GitSCM',poll:false,resetFirst: true, branches: [[name: "${VERSION_REGISTRY_BRANCH}"]], 
							doGenerateSubModuleConfiguration: false,							
							extensions: [ [$class: "CleanBeforeCheckout"], [$class: "CleanCheckout"], [$class: "RelativeTargetDirectory", releativeTargetDir: "${WORKSPACE}"], [$class: "WipeWorkspace"]],
							submoduleCfg: [],
							userRemoteConfigs: [[refspec: "+refs/heads/${VERSION_REGISTRY_BRANCH}:refs/remotes/origin/${VERSION_REGISTRY_BRANCH}", url: "${GIT_VR_URI}.git", credentialsId: "${GIT_CRDS}"]]
						]
						)}
					  dir('APPLICATIONREPO'){
					  checkout (
						scm: [
							$class: 'GitSCM',poll:false,resetFirst: true, branches: [[name: "${APPLICATIONREPO_BRANCH}"]], 
							doGenerateSubModuleConfiguration: false,							
							extensions: [ [$class: "CleanBeforeCheckout"], [$class: "CleanCheckout"], [$class: "RelativeTargetDirectory", releativeTargetDir: "${WORKSPACE}"], [$class: "WipeWorkspace"]],
							submoduleCfg: [],
							userRemoteConfigs: [[refspec: "+refs/heads/${APPLICATIONREPO_BRANCH}:refs/remotes/origin/${APPLICATIONREPO_BRANCH}", url: "${GIT_TP_URI}.git", credentialsId: "${GIT_CRDS}"]]
						]
						)}					
						currentBuild.description = "Pipeline type of execution: ${REQUESTEDACTION}"
                      }
                  }
              }
        }
 
        stage ("Develop: Unit Test and Build") {
            when {
                expression { params.REQUESTEDACTION == "devbuild" }
            }
            steps {
                script {
                    ansiColor("xterm") {
                        sh "cd ${WORKSPACE}/APPLICATIONREPO;mvn clean install"
                    }
                }
            }
        }
  
        stage ("Develop: Deploy to Nexus") {
            when {
                expression { params.REQUESTEDACTION == "devbuild" }
            }
            steps {
                script {
                    ansiColor("xterm") {
                        sh "cd ${WORKSPACE}/APPLICATIONREPO;mvn deploy"
                    }
                }
            }
        }
        
    } // end stages
} // end pipeline