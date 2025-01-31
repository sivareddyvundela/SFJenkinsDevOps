#!groovy

import groovy.json.JsonSlurperClassic

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTALS_ID=env.SERVER_KEY_CREDENTALS_ID
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL ?: "https://login.salesforce.com"

    def toolbelt = tool 'toolbelt'


    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }

    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
    
    withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Hub org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize DevHub') {
                rc = sh returnStatus: true, script: "${toolbelt}/sf org login jwt --client-id ${SF_CONSUMER_KEY} --jwt-key-file ${server_key_file} --username ${SF_USERNAME} --instance-url ${SF_INSTANCE_URL} --alias my-org --set-default"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Deploy Code Changes using Manifest File with Username 
            // -------------------------------------------------------------------------

            stage('Deploying Code to Org') {
                rc = sh returnStatus: true, script: "${toolbelt}/sf project deploy start --manifest manifest/package.xml --target-org ${SF_USERNAME}"
                if (rc != 0) {
                    error 'Salesforce org deployment failed.'
                }
            }
       }
    }
}

// Reference: https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_ci_jenkins_code.htm