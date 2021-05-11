@Library('eigi-jenkins-library') _

def pipeline = new ctct.v1.Pipeline(this)
def gitCmd = new common.v1.GitCmd(this)
def when = new common.v1.When(this)
def runWith = new common.v1.RunWith(this)
def log = new common.v1.Log(this)

pipeline.build() {
    stage('Clone repo') {
        gitCmd.checkout()
    }

    runWith.ruby {
      stage('Install') {
        sh(script: 'bundle install')
      } 
  
      stage('Version') {

        when.notBuildingPR {
          env.APP_VERSION = sh(returnStdout: true, script: 'rake bump:show-next INCREMENT=patch').trim()
        }

        when.buildingPR {
          log.info 'Setting version for PR build'
          def currentVersion = sh(returnStdout: true, 
            script: "rake bump:current | sed -ne 's/[^0-9]*\\(\\([0-9]\\.\\)\\{0,4\\}[0-9][^.]\\).*/\\1/p'").trim()
          env.APP_VERSION = "${currentVersion}-pr-${UUID.randomUUID().toString()}"
        }
        log.info "Version set to ${env.APP_VERSION} for this build"
      }

      stage('Build') {
        sh(script: 'rake')
      }
    }
    //parallel(javaProject.getScanners())
    // when.buildingMaster(){
    //     stage('Promote') {
    //         javaProject.promoteArtifacts(buildInfo)
    //     }
    //  }
}