pipeline {
    agent any

    // triggers {
    //     cron('H(0) 0 1 * *')
    // }

    environment {
        API_ENDPOINT        = 'https://api.cf.eu10.hana.ondemand.com'
        ORGANIZATION        = 'ISR Information Products AG_sap-im'
        CI_SPACE            = 'DEV'
        modulePaths         = sh(script: '''awk -F: '$1 ~ /path/ { gsub(/\\s/,"", $2); print $2 }' ${WORKSPACE}/mta.yaml''', returnStdout: true)
        mtaName             = sh(script: '''awk -F: '$1 ~ /^ID/ { gsub(/\\s/,"", $2); print $2 }' ${WORKSPACE}/mta.yaml''', returnStdout: true).trim()

        HANA_TECHN_CREDS    = credentials('sap-im-database-user-ti_cicd')
        HANA_TECHNICAL_USER       = "${HANA_TECHN_CREDS_USR}"
        HANA_TECHNICAL_PASSWORD   = "${HANA_TECHN_CREDS_PSW}"      
        
    }
    stages {
        stage('Build') {
			when {
                expression {env.BRANCH_NAME == 'release'}
            }
            steps {
                echo 'Building..'
                // create local npmrc file
                    sh('''cat <<EOF > .npmrc
                    registry=https://registry.npmjs.org/
                    @sap:registry=https://registry.npmjs.org
                    EOF''')

                // add .npmrc to the modulePaths
                    sh('''for path in ${WORKSPACE}/${modulePaths}; do
                        if [ -d "$path" ];
                            then ln -sft $path ../.npmrc
                        fi
                        done''')

                // build mtar package
                    sh('/var/jenkins_home/SAP/mbt/mbt build -t ./ --mtar ${mtaName}.mtar -m=verbose')
            }
        }
        stage('Deploy') {
			when {
                expression {env.BRANCH_NAME == 'release'}
            }        
            steps {
	                echo 'Deploying....'
	                sh('cf api $API_ENDPOINT --cacert /var/jenkins_home/SAP/xs_client/xsa_api.crt') // ohne Zertifikat: --skip-ssl-validation
	                sh('cf login -u $HANA_TECHNICAL_USER -p $HANA_TECHNICAL_PASSWORD -o $ORGANIZATION -s $CI_SPACE')
	                sh('cf deploy -f ${WORKSPACE}/mta_archives/${mtaName}.mtar')
                }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
        success {
            echo "Pipeline succeeded! Status: ${currentBuild.result}"
        }
        failure {
            echo "Pipeline failed! Status: ${currentBuild.result}"
            echo "Error: ${currentBuild.description}"
        }
    }   
}
