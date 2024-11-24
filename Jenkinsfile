pipeline {
    agent any

    stages {

        stage("Initial cleanup") {
              steps {
                dir("${WORKSPACE}") {
                  deleteDir()
              }
          }
        }

        stage('Checkout SCM') {
          steps {
                git branch: 'main', url: 'https://github.com/ksal1235/php-todo.git'
          }
        }

        stage('Prepare Dependencies') {
          steps {
                sh 'mv .env.sample .env'
                sh 'composer install'
                sh 'php artisan migrate'
                sh 'php artisan db:seed'
                sh 'php artisan key:generate'
          }
        }

        stage('Execute Unit Tests') {
          steps {
                sh './vendor/bin/phpunit'
          }
        }
        stage('Code Analysis') {
          steps {
            sh 'phploc app/ --log-csv build/logs/phploc.csv'

          }
        }
        stage('Plot Code Coverage Report') {
                steps {
                    plot csvFileName: 'plot-loc.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of Code', yaxis: 'Lines of Code'
                    plot csvFileName: 'plot-structures.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
                    plot csvFileName: 'plot-avg-length.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
                    plot csvFileName: 'plot-complexity.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'
                    plot csvFileName: 'plot-classes.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
                    plot csvFileName: 'plot-methods.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
                    plot csvFileName: 'plot-constants.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
                    plot csvFileName: 'plot-testing.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
                    plot csvFileName: 'plot-lloc.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
                    plot csvFileName: 'plot-functions.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
                    plot csvFileName: 'plot-objects.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'
          }
        }
        /*
        stage('SonarQube Quality Gate') {
                when {
                    expression {
                        def branchName = env.BRANCH_NAME ?: sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()
                        echo "Current branch: ${branchName}"
                        def match = branchName ==~ /^(develop|hotfix|release|main|master)$/
                        echo "Branch matches condition: ${match}"
                        return match
                    }
                }
                environment {
                    scannerHome = tool 'SonarQubeScanner'
                }
                steps {
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                    timeout(time: 5, unit: 'MINUTES') {
                        script {
                            def qg = waitForQualityGate()
                            echo "Quality Gate status: ${qg.status}"
                            echo "Quality Gate details: ${qg}"
                            // Add detailed logging of qg object
                            echo "Quality Gate details (full): ${qg.inspect()}"
                            if (qg.status != 'OK') {
                                echo "Quality Gate failed: ${qg.status}"
                                error "Pipeline aborted due to quality gate failure: ${qg.status}"
                            } else {
                                echo "Quality Gate passed: ${qg.status}"
                            }
                        }
                    }
                }
            }


        stage ('Package Artifact') {
          steps {
                  sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
          }
        }
        stage ('Upload Artifact to Artifactory') {
        steps {
            script {
                def server = Artifactory.server 'artifactory-server'
                def uploadSpec = """{
                    "files": [
                    {
                    "pattern": "php-todo.zip",
                    "target": "<name-of-artifact-repository>/php-todo",
                    "props": "type=zip;status=ready"

                    }
                    ]
                }"""
                println "Upload Spec: ${uploadSpec}"
                                  try {
                                      server.upload spec: uploadSpec
                                      println "Upload successful"
                                  } catch (Exception e) {
                                      println "Upload failed: ${e.message}"
                                    }
                                }
                              }
        }
        
*/
  }

}
