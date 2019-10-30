pipeline {
    agent {
        label 'agent_vkorchuk'
    }
    tools {
        nodejs 'Node12'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/korchuk/mdt-lab'
            }
        }
        stage('Sonar') {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube-external', credentialsId: 'sonarqube-server') {
                    script {
                        sonarHome = tool 'sonarscanner4'
                        sh """
                        ${sonarHome}/bin/sonar-scanner -Dsonar.projectKey=student23-project -Dsonar.sources=www
                        """
                    }
                }
            }

        }
        stage('Quality gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        stage('Build') {
            when {
                branch 'master'
            }
            parallel {
                stage('JS') {
                    steps {
                        sh label: 'minimize JS', script: """
                        cd ${WORKSPACE}/www/js
                        uglifyjs --timings init.js -o ../min/custom-min.js
                        """
                    }
                }
                stage('CSS') {
                    steps {
                        sh label: 'minimize CSS', script: """
                        cd ${WORKSPACE}/www/css
                        cleancss -d style.css > ../min/custom-min.css"""
                    }
                }
                stage('Prepare artifact') {
                    steps {
                        sh label: 'archive', script: """
                        cd ${WORKSPACE}/www
                        tar --exclude='./css' --exclude='./js' -c -z -f ../site-archive.tgz ."""
                    }
                }
                
            }
        }
        stage ('Nexus_upload') {
                    steps {
                        script {
                            def baseVersion = readFile 'version.txt'
                            nexusArtifactUploader artifacts: [[artifactId: 'site-archive', classifier: '${baseVersion}-${BUILD_NUMBER}', file: 'site-archive.tgz', type: 'tgz']], credentialsId: 'student23-nexus-key', groupId: 'site-archive', nexusUrl: 'master.jenkins-practice.tk:9443', nexusVersion: 'nexus3', protocol: 'https', repository: 'student23-repo', version: '${baseVersion}-${BUILD_NUMBER}'
                        }
                    }
        }
        post {
                success {
                    writeFile file: '../deploy_version', text: "${baseVersion}-${BUILD_NUMBER}"
                }
        }
    }
}
