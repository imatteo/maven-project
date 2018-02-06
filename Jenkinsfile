pipeline {
    agent any // use any jenkins host available

    parameters { // parameters directive
        // we define two variables containing the IP address of the two tomcat instances.
        // Best practice is to use parameters instead of hardcoded values in our scripts.
        string(name: 'tomcat_dev', defaultValue: 'localhost', description: 'Staging Server')
        string(name: 'tomcat_prod', defaultValue: 'localhost', description: 'Production Server')
    } 

    triggers { // triggers directive
        // we specify that we are going to poll the repo every 1 minute
        pollSCM('* * * * *');
    } 

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
            post { // post conditions
                success { // if success
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war' // it grabs built artifacts e copy them to the specified location. Jenkins’s built-in support for storing "artifacts", files generated during the execution of the Pipeline.
                }
            }
        }

        stage ('Deployments') {
            parallel { // it tells Jenkins that it is possible to run the two following stages at the same time.

                stage ('Deploy to Staging') {
                    steps {
                        sh "sshpass -p 'tibco123' scp **/target/*.war catenate@${params.tomcat_dev}:/opt/tomcat/apache-tomcat-9.0.4-staging/webapps"
                        // AWS example:
                        // sh "scp -i /home/jenkins/tomcat-demo.pem **/target/*.war ec2-user@${params.tomcat_dev}:/var/lib/tomcat7/webapps"
                    }
                    post {
                        success {
                            echo 'Code deployed to Dev.'
                        }

                        failure {
                            echo ' Deployment failed.'
                        }
                    }
                }

                stage ('StaticAnalysis') {
                    sh 'mvn checkstyle:checkstyle'
                } 
                
            } 
        } 

        stage ("Deploy to Production") {
            steps {
                timeout(time:5, unit:'DAYS'){ // sets a timeout period for the Pipeline run, after which Jenkins should abort the Pipeline (5 days in this case).
                    input message:'Approve PRODUCTION Deployment?'
                }
                sh "sshpass -p 'tibco123' scp **/target/*.war catenate@${params.tomcat_dev}:/opt/tomcat/apache-tomcat-9.0.4-production/webapps"
            }
            post {
                success {
                    echo 'Code deployed to Production.'
                }

                failure {
                    echo ' Deployment failed.'
                }
            }
        }
    }  
} 