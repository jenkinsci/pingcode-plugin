if (JENKINS_URL == 'https://ci.jenkins.io/') {
    def configurations = [
        [ platform: "linux", jdk: "8", jenkins: null ],
    ]
    buildPlugin(configurations: configurations, timeout: 180, useContainerAgent: true)
    return
}

def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, cloud: 'kubernetes',
    containers: [
        containerTemplate(name: 'wtctl', image: 'registry.cn-beijing.aliyuncs.com/worktile/wtctl:latest', alwaysPullImage: true, ttyEnabled: true, command: 'cat'),
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock'),
        hostPathVolume(mountPath: '/root/.ssh', hostPath: '/root/.ssh'),
        hostPathVolume(mountPath: '/tmp/cache', hostPath: '/date/cache/pingcode-plugin')
    ]
) {
    node(label) {

        def scmVars = checkout scm

        def commit = scmVars.GIT_COMMIT
        def branch = scmVars.GIT_BRANCH

        stage('Using PingCode Pipeline') {
            script {

                if (env.DISABLE_WTCTL != "true") {
                    if (env.RUN_BETA != "true") {
                        container('wtctl') {
                            sh 'wtctl'
                        }
                   }

                if (env.RUN_BETA == "true") {
                    echo 'Using PingCode Pipeline'
                    container('wtctl') {
                        sh 'export TAG_NAME=$(git describe --tags) && wtctl'
                    }
                   }
                }
            }
        }
    }
 }
