pipeline {
    agent any

    stages {
      stage('Build') {
            steps {
                echo 'Building..'
                sh 'docker build -t workshop-shoppingcart-api-test .'
            }
        }
        stage('Run Unit and Integrate Test') {
            parallel {
                stage('Unit Test') {
                    steps {
                        echo 'Unit Testing..'
                        sh 'docker run --rm -e RUNNING_PROJECT=./tests/api.UnitTest/api.UnitTest.csproj workshop-shoppingcart-api-test'
                    }
                }
                stage('Integrate Test') {
                    steps {
                        echo 'Integrate Testing..'
                        sh 'docker run --rm -e RUNNING_PROJECT=./tests/api.IntegrationTest/api.IntegrationTest.csproj workshop-shoppingcart-api-test'
                    }
                }
            }
        }
        stage('Setup Integrate Test Environment') {
            steps {
                
                echo 'UI Integrate Testing....'
                echo '# Start Database Server'
                echo '## Build database docker image'
                sh 'docker build -t workshop-shoppingcart-mysql . -f Dockerfile_mysql'

                echo '## Run container'
                sh 'docker run --rm -d --name=workshop-shoppingcart-mysql -p 3306:3306 workshop-shoppingcart-mysql'

                sh 'sleep 15'

                echo '# Data Migration into Mysql'
                echo '## run docker liquibase\'s image to migrate data from changelog.yml'
                

                dir("data/Production") {
                    script {
                        def workspace = pwd()
                        def myVar = "${env.BASE_PATH}"

                        def outter_docker_workspace = workspace.replace("/var/jenkins_home",myVar)

                        sh "docker run --rm -v $outter_docker_workspace:/liquibase/ -e \"LIQUIBASE_URL=jdbc:mysql://docker.for.mac.localhost/workshop_shoppingcart\" -e \"LIQUIBASE_USERNAME=root\" -e \"LIQUIBASE_PASSWORD=1234\" -e \"LIQUIBASE_CHANGELOG=/liquibase/changelog.yml\" webdevops/liquibase:mysql update"
                    }
                }

                echo '# Data Migration for ui.AcceptanceTest'
                echo '## run docker liquibase\'s image to migrate data from changelog.yml'
                

                dir("data/UserAcceptanceTest") {
                    script {
                        def workspace = pwd()
                        def myVar = "${env.BASE_PATH}"

                        def outter_docker_workspace = workspace.replace("/var/jenkins_home",myVar)

                        sh "docker run --rm -v $outter_docker_workspace:/liquibase/ -e \"LIQUIBASE_URL=jdbc:mysql://docker.for.mac.localhost/workshop_shoppingcart\" -e \"LIQUIBASE_USERNAME=root\" -e \"LIQUIBASE_PASSWORD=1234\" -e \"LIQUIBASE_CHANGELOG=/liquibase/changelog.yml\" webdevops/liquibase:mysql update"
                    }
                }
                echo '# Install API'

                echo '## Build Image'
                dir("src/api/") {
                    sh 'docker build -t workshop-shoppingcart-api .'
                }

                echo '## Run container'
                
                sh 'docker run --rm -d --name workshop-shoppingcart-api -p 5001:5001 -e ConnectionString="server=docker.for.mac.localhost;userid=root;password=1234;database=workshop_shoppingcart;convert zero datetime=True;CHARSET=utf8;" workshop-shoppingcart-api'

                echo '# Install UI'
                echo '## Build Image'
                
                dir("src/ui/") {
                    sh 'docker build -t workshop-shoppingcart-ui .'
                }


                echo '## Run container'
                sh 'docker run --rm -d --name workshop-shoppingcart-ui -p 80:80 workshop-shoppingcart-ui'

                echo '# Run Robot Framework'
            }
        }
        stage('Run UI Integrate Test') {
            parallel {
                stage('Test On Chrome') {
                    steps {
                        dir("tests/ui.AcceptanceTest/") {
                            script {
                                def workspace = pwd()
                                def myVar = "${env.BASE_PATH}"

                                def outter_docker_workspace = workspace.replace("/var/jenkins_home",myVar)
            
                                sh "docker run --rm -v $outter_docker_workspace/chrome/reports:/opt/robotframework/reports -v $outter_docker_workspace:/opt/robotframework/tests -e ROBOT_OPTIONS=\"--exclude 'Not Implement' --variable URL:http://docker.for.mac.localhost --variable BROWSER:firefox\" siamchamnankit/sck-robot-framework"
                            }
                        }
                    }
                }
                stage('Test On Firefox') {
                    steps {
                        dir("tests/ui.AcceptanceTest/") {
                            script {
                                def workspace = pwd()
                                def myVar = "${env.BASE_PATH}"

                                def outter_docker_workspace = workspace.replace("/var/jenkins_home",myVar)

                                sh "docker run --rm -v $outter_docker_workspace/firefox/reports:/opt/robotframework/reports -v $outter_docker_workspace:/opt/robotframework/tests -e ROBOT_OPTIONS=\"--exclude 'Not Implement' --variable URL:http://docker.for.mac.localhost --variable BROWSER:firefox\" siamchamnankit/sck-robot-framework"
                            }
                        }                        
                    }
                }
                stage('Test On Safari') {
                    steps {
                        dir("tests/ui.AcceptanceTest/") {
                            script {
                                def workspace = pwd()
                                def myVar = "${env.BASE_PATH}"

                                def outter_docker_workspace = workspace.replace("/var/jenkins_home",myVar)

                                sh "docker run --rm -v $outter_docker_workspace/safari/reports:/opt/robotframework/reports -v $outter_docker_workspace:/opt/robotframework/tests -e ROBOT_OPTIONS=\"--exclude 'Not Implement' --variable URL:http://docker.for.mac.localhost --variable BROWSER:firefox\" siamchamnankit/sck-robot-framework"
                            }
                        }                        
                    }
                }
            }
        }
        stage('Stop Integrate Test Environment') {
            steps {
                echo 'Stop Integrate Testing....'
                echo '# Stop UI Server'
                sh 'docker stop workshop-shoppingcart-ui'
                echo '# Stop API Server'
                sh 'docker stop workshop-shoppingcart-api'
                echo '# Stop Database Server'
                sh 'docker stop workshop-shoppingcart-mysql'
            }

        }
        stage('UAT Deploy') {
            steps {
                echo 'UAT Deploying....'
            }
        }
    }
}
