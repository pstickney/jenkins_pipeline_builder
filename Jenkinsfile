@Library('eigi-jenkins-library') _

def pipeline = new ctct.v1.Pipeline(this)
def gitCmd = new common.v1.GitCmd(this)
def when = new common.v1.When(this)
def runWith = new common.v1.RunWith(this)
def log = new common.v1.Log(this)
def artifactory = new helpers.common.v1.CommonArtifactoryWrapper(this)
def configManager = new helpers.common.v1.CommonConfigManager(this)

pipeline.build() {
  stage('Clone repo') {
    gitCmd.checkout()
  }

  runWith.ruby {
    stage('Install') {
      sh(script: 'bundle install')
    } 

    stage('Version') {
      when.buildingMaster {
        env.APP_VERSION = sh(returnStdout: true, script: 'rake bump:show-next INCREMENT=patch').trim()
        log.info "Version set to ${env.APP_VERSION} for this build"
        
        sh(script: 'rake bump:patch')
        
        gitCmd.run("git tag -a ${env.APP_VERSION} -m 'Release ${env.APP_VERSION}'")
        String pushCommitCmd = "git push origin ${env.BRANCH_NAME}"
        gitCmd.run(pushCommitCmd)

        String tagCmd = "git push origin --tags"
        gitCmd.run(tagCmd)
      }

      when.buildingPR {
        log.info 'Setting version for PR build'
        def currentVersion = sh(returnStdout: true, 
          script: "rake bump:current | sed -ne 's/[^0-9]*\\(\\([0-9]\\.\\)\\{0,4\\}[0-9][^.]\\).*/\\1/p'").trim()
        env.APP_VERSION = "${currentVersion}-pr-${UUID.randomUUID().toString()}"
        log.info "Version set to ${env.APP_VERSION} for this build"
      }
    }

    stage('Build') {
      sh(script: 'rake')
    }

    stage('Publish') {
      // no idea why this wasn't working through the configmap
      configManager.setConfigProperty('artifactoryCredentialId', 'buildmaster_ad_creds')
      String repoTarget = ""
      String uploadSpec = """{
        "files": [
          {
            "pattern": "*.gem",
            "target": "${repoTarget}",
            "props": "build.name=${env.JOB_NAME}"
          }
        ]
      }"""

      when.buildingPR {
        repoTarget = "gems-stage/gems"
        artifactory.uploadArtifact(uploadSpec, false)
      }
      
      when.buildingMaster {
        repoTarget = "gems-stage/gems"
        // repoTarget = "gems-local/gems"
        artifactory.uploadArtifact(uploadSpec, false)
      }
    }
  }
}