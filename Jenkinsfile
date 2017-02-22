#!groovy

def DEV_EmailRecipients='amar.tyagi@kelltontech.com'
def QA_EmailRecipients='pratap.hada@kelltontech.com'
def MNGR_EmailRecipients='vijay.kumar@kelltontech.com'

def QA_BuildAuthorization='amar.tyagi@kelltontech.com'
def PROD_BuildAuthorization='vijay.kumar@kelltontech.com'


// Constants variable used below for msg
def BUILD_SUCCESS_AFTER_FAILED='Hi, Build process completed successfully after failure last build!'
def BUILD_FAILED='Hi, Build process failed! Check attached log file.'
def BUILD_PUBLISH_QA_STAGE_SUCCESS='Hi, Build successfully published to given recipients.'
def BUILD_PUBLISH_FAILED='Hi, Build publish failed, please check attached log file.'

node() {
    echo"environment "+env.toString()
    String branchName = env.BRANCH_NAME
    echo" branch name "+branchName
    echo" GIT COMMIT name "+env.GIT_COMMIT
    gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
    echo" valueee "+gitCommit
    try {
            stage ('Checkout'){
              checkout scm
            }

            stage ('Build')
            {
                // todo one time
                sh 'chmod a+x ./gradlew'
                sh './gradlew clean lint assemble'

                androidLint canComputeNew: false, defaultEncoding: '', healthy: '', pattern: '**/lint-results*.xml', unHealthy: ''
                if(currentBuild.previousBuild!=null && currentBuild.previousBuild.result!=null && !currentBuild.previousBuild.result.toString().equals('SUCCESS'))
                {
                     sendEmails(DEV_EmailRecipients,BUILD_SUCCESS_AFTER_FAILED,'',false)
                }
            }

            currentBuild.result='SUCCESS'
        }
        catch (Exception e)
        {
            currentBuild.result='FAILURE'
            echo" failure exception "+e.toString()
            sendEmails(DEV_EmailRecipients,BUILD_FAILED,'',true)
        }

        if( currentBuild.result=='SUCCESS') {
            try {
                stage('Publish')
                        {
                            if (branchName == 'develop' || branchName.startsWith('feature')) {
                                timeout(time: 60, unit: 'SECONDS')
                                        {
                                           /* def outcome = input id: 'Want to email build?',
                                                    message: 'Send Build?',
                                                    ok: 'Okay',
                                                    parameters: [
                                                            [
                                                                    $class: 'ChoiceParameterDefinition', choices: 'select\nYes\nNo',
                                                                    name: 'Take your pick',
                                                                    description: ''
                                                            ]
                                                    ]

                                            echo" ans "+outcome
                                            if("Yes".equals(outcome))*/
                                            input message: 'Want to email build to developer?'
                                                sendEmails(DEV_EmailRecipients,BUILD_PUBLISH_QA_STAGE_SUCCESS, '**/*debug*.apk', false)
                                        }
                            } else if (branchName.startsWith('release')) {
                                timeout(time: 60, unit: 'SECONDS')
                                        {
                                            def outcome = input id: 'Want to email build?',
                                                    message: 'Please select your choice?',
                                                    ok: 'Okay',
                                                    parameters: [
                                                            [
                                                                    $class: 'ChoiceParameterDefinition', choices: 'select\nStage\nProduction\nBoth',
                                                                    name: 'Take your pick',
                                                                    description: ''
                                                            ],
                                                            [
                                                                    $class: 'StringParameterDefinition',
                                                                    defaultValue: "",
                                                                    name: 'Enter emailID to receive build!',
                                                                    description: 'Multiple email id must be separated by single space'
                                                            ]
                                                    ]

                                            echo" ans "+outcome
                                            if("Stage".equals(outcome.get('Take your pick')))
                                                sendEmails(DEV_EmailRecipients+" "+outcome.get('Enter emailID to receive build!'),BUILD_PUBLISH_QA_STAGE_SUCCESS, '**/*stage*.apk', false)
                                            else if ("Production".equals(outcome.get('Take your pick')))
                                            sendEmails(DEV_EmailRecipients+" "+outcome.get('Enter emailID to receive build!'),BUILD_PUBLISH_QA_STAGE_SUCCESS, '**/*release*.apk', false)
                                            else if("Both".equals(outcome.get('Take your pick')) )
                                                sendEmails(DEV_EmailRecipients+" "+outcome.get('Enter emailID to receive build!'),BUILD_PUBLISH_QA_STAGE_SUCCESS, '**/*stage*.apk,**/*release*.apk', false)
                                        }

                            }

                        }
            }
            catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException ee) {
                echo(" timeout here ! build not published. ")
            }
            catch (Exception e) {
                echo" publish exception "+e.toString()
                sendEmails(DEV_EmailRecipients, BUILD_PUBLISH_FAILED, '', true)
            }
        }

    }

    def sendEmails(emailRecipient,msg,pattern,logAttach) {
        emailext attachLog: logAttach,body: msg,attachmentsPattern:pattern, subject: '$PROJECT_NAME - Build # $BUILD_NUMBER -'+currentBuild.result, to:emailRecipient
    }

  // stage ('upload')
  //      {
  //          try
  //          {
  //          androidApkUpload apkFilesPattern: '**/*.apk', googleCredentialsId: 'AmarExample', recentChangeList: [[language: 'es-US', text: 'New changes.']], trackName: 'alpha'
  //          }
  //          catch(Exception e)
  //          {
  //          sendEmails('''Hi,build not uploaded on play store , please see logs...''' +e.getMessage())
  //          }
  //      }*/

