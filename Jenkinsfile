pipeline {
    agent { 
        label 'Developer30' 
    }
    stages {
        stage('Test RDFUnit') {
            agent {
                docker { image 'aksw/rdfunit'
                    args '--entrypoint=""'}
            }
            steps {
                sh 'java -jar /app/rdfunit-validate.jar -d ./MatVoc-Core.ttl -f /tmp/ -o json-ld -s owl,rdfs'
                sh 'cp /tmp/results/._MatVoc-Core.ttl.aggregatedTestCaseResult.jsonld ./RDFUnit_results.jsonld'
            }
        }
        stage('Test OOPS') {
            agent {
                docker { image 'tboonx/oops_caller:0.2'
                    args '--entrypoint=""'}
            }
            steps {
                sh '/bin/sh /script.sh MatVoc-Core > oops_result.xml'
                sh 'ls -hal'
            }
        }
        stage('Interprete reports') {
            agent {
                docker { image 'alpine/xml'}
            }
            steps {
                sh 'sh -c " cat ./RDFUnit_results.jsonld | jq -c \'.[\\"@graph\\"] | .[] | select(.resultStatus | . and contains (\\"rut:ResultStatusFail\\"))\'  | jq . " > RDFUnit_errors.txt'
                sh './interprete.sh'
            }
        }
    }
    post {
        always {
            emailext attachmentsPattern: '*RDFUnit_errors.txt',
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}<br> More info at: <a href=\"${env.BUILD_URL}\">${env.BUILD_URL}</a><br><br>If your commit failed the job, then the RDFUnit report is attached.<br>Additionally the OOPS summary: (always the occurences amount and then the description)<br>Kritisch:<br><pre>${FILE, path='*critical.txt'}</pre><br>Important:<br><pre>${FILE, path='*important.txt'}</pre><br>Minor:<br><pre>${FILE, path='*minor.txt'}</pre><br><br>For more information read the job result.",
                recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']],
                subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
        }
    }
}
