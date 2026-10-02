def call(Map config = [:]) {
    if (!config.buildCommand || !config.testCommand) {
        error('Configurar buildCommand y testCommand')
    }

    pipeline {
        agent any

        options {
            disableConcurrentBuilds(abortPrevious: true)
        }

        stages {
            stage('Prepare') {
                steps {
                    script {
                        def subject = sh(
                            script: 'git log -1 --format=%s',
                            returnStdout: true
                        ).trim()

                        env.SKIP_PIPELINE = subject
                            .contains('chore: jenkins bump version')
                            .toString()

                        if (config.versionCommand) {
                            env.VERSION = sh(
                                script: config.versionCommand,
                                returnStdout: true
                            ).trim()
                        }
                    }
                }
            }

            stage('Install dependencies') {
                when {
                    expression {
                        !env.SKIP_PIPELINE.toBoolean() && config.installCommand
                    }
                }
                steps {
                    script { sh config.installCommand }
                }
            }

            stage('Build') {
                when { expression { !env.SKIP_PIPELINE.toBoolean() } }
                steps {
                    script { sh config.buildCommand }
                }
            }

            stage('Test') {
                when { expression { !env.SKIP_PIPELINE.toBoolean() } }
                steps {
                    timeout(time: 30, unit: 'MINUTES') {
                        script { sh config.testCommand }
                    }
                }
            }

            stage('SonarQube Analysis') {
                when {
                    expression {
                        !env.SKIP_PIPELINE.toBoolean() &&
                        (config.sonarCommands ?: []).size() > 0
                    }
                }
                steps {
                    script {
                        config.sonarCommands.each { command ->
                            withSonarQubeEnv(config.sonarInstallation) {
                                sh command
                            }
                            timeout(time: 5, unit: 'MINUTES') {
                                waitForQualityGate abortPipeline: true
                            }
                        }
                    }
                }
            }
        }

        post {
            always {
                echo "Resultado: ${currentBuild.currentResult}"
                cleanWs()
            }
        }
    }
}

---

@Library('mi-library') _

proyectoBuild(
    versionCommand: 'node -p "require(\'./package.json\').version"',
    installCommand: 'npm ci',
    buildCommand: 'npm run build',
    testCommand: 'npm test -- --run',
    sonarInstallation: 'sonar.roshka.com',
    sonarCommands: [
        'sonar-scanner -Dsonar.projectKey=mi-proyecto'
    ]
)