properties([
  buildDiscarder(logRotator(numToKeepStr: '20')),
  disableConcurrentBuilds(abortPrevious: true)
])

def runTests(Map params = [:]) {
  return {
    def agentContainerLabel = 'maven-' + params['jdk']
    if (params['platform'] == 'windows') {
      agentContainerLabel += '-windows'
    }
    boolean publishing = params['jdk'] == 21 && params['platform'] == 'linux'
    node(agentContainerLabel) {
      timeout(time: 1, unit: 'HOURS') {
        def stageIdentifier = params['platform'] + '-' + params['jdk']
        stage("Checkout (${stageIdentifier})") {
          checkout scm
        }
        stage("Build (${stageIdentifier})") {
          ansiColor('xterm') {
            infra.withArtifactCachingProxy(true) {
              def args = ['-Dstyle.color=always', '-Dmaven.test.failure.ignore', 'clean', 'install']
              if (publishing) {
                args += '-Dset.changelist'
              }
              // Needed for correct computation of JenkinsHome in RunMojo#execute.
              withEnv(['JENKINS_HOME=', 'HUDSON_HOME=']) {
                infra.runMaven(args, params['jdk'], null, null, false)
              }
            }
          }
        }
        stage("Archive (${stageIdentifier})") {
          if (publishing) {
            infra.prepareToPublishIncrementals()
          }
        }
      }
    }
  }
}

parallel(
    'linux-21': runTests(platform: 'linux', jdk: 21),
)
infra.maybePublishIncrementals()
