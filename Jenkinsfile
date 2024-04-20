def ENV_SELECT(condition) {
    if (condition == true){
        println("Prod Select")
        CLOUDSDK_CORE_PROJECT='comet-prod-c4c7'
        CLIENT_EMAIL='terraforon@erviceaccount.com '
        withCredentials([file(credentialsId: 'comet-prod', variable: 'GC_KEY')]) {
            sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
        }
    }else{
        CLOUDSDK_CORE_PROJECT='comet-nonprod-200617'
        CLIENT_EMAIL='jenkins@erviceaccount.com'
        //withCredentials([file(credentialsId: 'comet-nonprod', variable: 'GC_KEY')]) {
        //    sh("gcloud auth activate-service-account --key-file=${GC_KEY}")
        //}
        sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
        println("Dev Select")
    }
}


pipeline {
  agent any

  environment{
      BRANCH_NAME_TAG="${GIT_BRANCH.split("/").size() > 1 ? GIT_BRANCH.split("/")[1] : GIT_BRANCH}"
      BRANCH_NAME_TARGET="${env.gitlabTargetBranch}"
      ACTION_TYPE="${env.gitlabActionType}"
      SOURCE_BRANCH_NAME="${env.gitlabSourceBranch}"
      GITLAB_URL="${env.gitlabSourceRepoHttpUrl}"
      //MS_NAME="${env.gitlabSourceRepoName.toLowerCase()}"
      MS_NAME="postpaid-operation-flow-manager"
      STATUS="${env.gitlabMergeRequestState}"
      MICROSERVICE_NAME="${env.gitlabSourceRepoName}"
      APPLICATION_RUNNING_PORT= "${APPLICATION_RUNNING_PORT}"
      INITIAL="${INITIAL}"
      GITLAB_Checkout_Sha="${env.gitlabAfter}"
      GITLAB_SSH_URL="${env.gitlabSourceRepoSshUrl}"
      SCANNERHOME= tool 'SonarQubeScanner'
      SYSDIG_ENDPOINT = "https://us2.app.sysdig.com"
  }





  stages {


      stage('CleanWorkspace Before') {
          steps {

              step([$class: 'WsCleanup'])
              checkout scm
                      sh "ls -ltr"

          }


      }



      stage('Set Variables') {
          steps {
              println "Action Type is : ${ACTION_TYPE}"
              println "Target Branch Name is :${BRANCH_NAME_TARGET}"
              println "Source Branch Name is :${env.gitlabSourceBranch}"
              println "Micro Service Name is :${MS_NAME}"
              println "Status Name is :${STATUS}"
              println "MICROSERVICE_NAME= $MICROSERVICE_NAME"
              println "GITLAB_URL= $GITLAB_URL"
              println "Gitlab Checkout Sha = $GITLAB_Checkout_Sha"
              println "Gitlab SSH URL = $GITLAB_SSH_URL"
              println " Tag Created Branch Name :${env.BRANCH_NAME_TAG}"

              sh "echo ${GIT_COMMIT} | cut -c 1-8 > commit_number.txt"
              script{
                      COMMIT = readFile 'commit_number.txt'
                      COMMIT="${COMMIT}".trim()
                  }
          }
      }


      stage('Set The Docker TAG') {
          steps {
              script{
                  if ((env.gitlabTargetBranch == 'sit') && (env.gitlabActionType == 'PUSH'))
                  {
                      println "This is Push ro SIT Need to deploy on Dev"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:dev.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492//non-prod/dev/"
                      ENV= "development"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      DOCKER_IMAGE_TAG_BUAT = null
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  }else if ((env.gitlabTargetBranch == 'buat') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to Master  Need to deploy on BUAT"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "buat"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-mvs"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  }else if ((env.gitlabTargetBranch == 'staging') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to staging  Need to deploy on staging"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:stag.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/stag/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "staging"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-postmvs"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  //}else if (env.BRANCH_NAME_TAG == 'master' && env.gitlabActionType == 'TAG_PUSH')
                    }
                    else if ((env.gitlabTargetBranch == 'postpaid') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to postpaid  Need to deploy on staging on comet-postpaid namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:postpaid.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/postpaid/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "postpaid"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-postpaid"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  //}else if (env.BRANCH_NAME_TAG == 'master' && env.gitlabActionType == 'TAG_PUSH')
                    }
                   //else if ((env.BRANCH_NAME_TAG == 'master') && (env.gitlabActionType == 'TAG_PUSH'))
                   else if (env.BRANCH_NAME_TAG == 'master')
                  {
                      println "This is TAG to Branch Need to deploy IN Production"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-prod-c4c7/${MS_NAME}:prod.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7494/postpaid-operation-flow-manager/prod/"
                      ENV= "production"
                      ENV_BUAT= "buat"
                      setGcloudProd= true
                      DOCKER_IMAGE_TAG_BUAT = null
                      println "Change the GCP Account to Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account terraform-automation@comet-prod-c4c7.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.BRANCH_NAME_TAG}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  }

                   else if ((env.gitlabTargetBranch == 'genprio-soa-sit') && (env.gitlabActionType == 'PUSH'))
                  {
                      println "This is Push to genprio-soa-sit  Need to deploy on staging on genprio-soa-sit namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:genprio-soa-sit.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/genprio-soa-sit/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "genprio-soa-sit"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-genprio-soa-sit"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"

                    }
                    else if ((env.gitlabTargetBranch == 'genprio-soa-staging') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to genprio-soa-staging  Need to deploy on staging on genprio-soa-staging namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:genprio-soa-staging.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/genprio-soa-staging/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "genprio-soa-staging"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-genprio-soa-staging"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"

                    }else if ((env.gitlabTargetBranch == 'xlcenter-sit') && (env.gitlabActionType == 'PUSH'))
                  {
                      println "This is Push to xlcenter-sit  Need to deploy on staging on xl-center-sit namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:xlcenter-sit.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/xlcenter-sit/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "xlcenter-sit"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-xlcenter-sit"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"

                    }
                    
                    else if ((env.gitlabTargetBranch == 'xlcenter-staging') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                    //else if ((env.gitlabTargetBranch == 'xlcenter-staging') && (env.gitlabActionType == 'PUSH'))
                  {
                      println "This is Merge to xlcenter-staging  Need to deploy on staging on xl-center-staging namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:xlcenter-staging.v.$COMMIT"
                      DOCKER_IMAGE_TAG_BUAT= "gcr.io/comet-nonprod-200617/${MS_NAME}:buat.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/xlcenter-staging/"
                      GCP_BUCKET_BUAT="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/buat/"
                      ENV= "xlcenter-staging"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-xlcenter-staging"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"

                    }
                    
// XL TEAM CICD -------------------------------------------------------------------->
                    else if ((env.gitlabTargetBranch == 'sit-xlscid') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to sit-xlscid  Need to deploy on sit-xlscid namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:sit-xlscid.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/sit-xlscid/"
                      ENV= "sit-xlscid"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      DOCKER_IMAGE_TAG_BUAT = null
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-sit-xlscid"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  }

                  else if ((env.gitlabTargetBranch == 'staging-xlscid') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to staging-xlscid  Need to deploy on staging-xlscid namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:staging-xlscid.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/staging-xlscid/"
                      ENV= "staging-xlscid"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      DOCKER_IMAGE_TAG_BUAT = null
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-staging-xlscid"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  }
                  else if ((env.gitlabTargetBranch == 'sit-xlwebsupport') && (env.gitlabActionType == 'MERGE') && (env.gitlabMergeRequestState == 'merged'))
                  {
                      println "This is Merge to staging-xlscid  Need to deploy on sit-xlwebsupport namespace"
                      DOCKER_IMAGE_TAG = "gcr.io/comet-nonprod-200617/${MS_NAME}:sit-xlwebsupport.v.$COMMIT"
                      GCP_BUCKET="gs://kuberneties-manifest-7492/postpaid-operation-flow-manager/non-prod/sit-xlwebsupport/"
                      ENV= "sit-xlwebsupport"
                      ENV_BUAT= "buat"
                      setGcloudProd= false
                      DOCKER_IMAGE_TAG_BUAT = null
                      println "Change the GCP Account to Non-Production"
                      sh "gcloud auth list"
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
                      skipRemainingStages = false
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                      env.SONARID="${MS_NAME}-sit-xlwebsupport"
                      println "sonarid=${SONARID}"
                      ADL_MIRROR_BRANCH="${env.gitlabTargetBranch}"
                      println "code will be mirrored to ADL ${MS_NAME} : ${ADL_MIRROR_BRANCH} branch"
                  }


                  else
                  {
                      println "Not Match Any Pipeline Condition"
                      skipRemainingStages = true
                      skipBUATSteps = true
                      println "skipRemainingStages = ${skipRemainingStages}"
                  }


              }


          }

      }



      stage('Cloning our Git') {

          when {
              expression {
                  !skipRemainingStages
              }
          }
          steps {
              script {


                  //if ((env.BRANCH_NAME_TAG == 'master') && (env.gitlabActionType == 'TAG_PUSH'))
                  //{
                      def ssh_url= env.GITLAB_SSH_URL

                      checkout([$class: 'GitSCM',
                  branches: [[name: "${env.gitlabSourceBranch}"]],
                  doGenerateSubmoduleConfigurations: false,
                  extensions: [], submoduleCfg: [],
                  userRemoteConfigs: [[credentialsId: scm.getUserRemoteConfigs()[0].getCredentialsId() , url: ssh_url]]])

                      sh "ls -ltr"
                      sh "git remote rename origin upstream"
                      //sh "git remote add origin https://sudaraka:SUD%40%2112260889988@gitlab.axiatadigitallabs.com/sudaraka-devops/mirror_test.git"
                      sh "git remote add origin https://oauth2:Sj64AjB9aerYEqsiKTbg@gitlab.axiatadigitallabs.com/comet-ui/et-telco-xl-comet-postpaidoperationflowmanager.git"
                      sh "git checkout -b ${ADL_MIRROR_BRANCH}"
                      println "commit success"
                      sh "git push -f -u origin ${ADL_MIRROR_BRANCH}"
                      println "after clone : ${GIT_BRANCH}"

                  //}else
                  //{
                  //    println "Mirroring will not occur"
                  //    skipRemainingStages = false
                  //    skipBUATSteps = true
                  //    println "skipRemainingStages = ${skipRemainingStages}"
                  //}

                  //def ssh_url= env.GITLAB_SSH_URL

                  //checkout([$class: 'GitSCM',
                  //branches: [[name: "${env.gitlabSourceBranch}"]],
                  //doGenerateSubmoduleConfigurations: false,
                  //extensions: [], submoduleCfg: [],
                  //userRemoteConfigs: [[credentialsId: scm.getUserRemoteConfigs()[0].getCredentialsId() , url: ssh_url]]])

                  //sh "ls -ltr"
                  //sh "git remote rename origin upstream"
                  //sh "git remote add origin https://sudaraka:SUD%40%2112260889988@gitlab.axiatadigitallabs.com/sudaraka-devops/mirror_test.git"
                  //sh "git checkout -b master"
                  //println "commit success"
                  //sh "git push -u origin master"

              }
          }

      }




      stage('Build') {
          when {
              expression {
                  !skipRemainingStages
              }
          }


          steps {
              script{
                      sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"  
                      DOCKER_IMAGE = docker.build("postpaid-operation-flow-manager:latest", "-f Dockerfile .")
                      sh "docker images"
                      sh "docker create -ti --name postpaid-operation-flow-manager-$BUILD_NUMBER postpaid-operation-flow-manager:latest bash"
                      sh "docker cp postpaid-operation-flow-manager-$BUILD_NUMBER:/usr/src/app/target ."
                      sh "ls target/"
                      sh "docker rm -f postpaid-operation-flow-manager-$BUILD_NUMBER"
                      sh "docker ps -a"

              }
          }
      }







      stage('Yaml Create') {

          when {
              expression {
                  !skipRemainingStages
              }
          }
          steps {
            script {

              if ( setGcloudProd )
              {
                //sh "gcloud config set account terraform-automation@comet-prod-c4c7.iam.gserviceaccount.com"
                ENV_SELECT(true)
              }
              else
              {
                 sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
              }

              sh "bash -c \" source yamlcreate.sh ${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_TAG_BUAT} ${ENV} ${ENV_BUAT}\""
              sh "gsutil cp ${ENV}/${MS_NAME}.yaml ${GCP_BUCKET}"
              if ( !skipBUATSteps ){
                 sh "gsutil cp ${ENV_BUAT}/${MS_NAME}.yaml ${GCP_BUCKET_BUAT}"
                  }
              }
          }
      }


      stage('Docker Build') {
          when {
              expression {
                  !skipRemainingStages
              }
          }
          steps {

              script{

                  sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"  
                  DOCKER_IMAGE = docker.build("${DOCKER_IMAGE_TAG}", "-f ${ENV}/Dockerfile .")
                  println "new dockerimg" + " " + DOCKER_IMAGE.id

                  if ( !skipBUATSteps ){
                  DOCKER_IMAGE_BUAT = docker.build("${DOCKER_IMAGE_TAG_BUAT}", "-f ${ENV_BUAT}/Dockerfile .")
                  println "new dockerimg" + " " + DOCKER_IMAGE_BUAT.id
                  }



              }

          }
      }



      stage('Docker push') {
          steps {
              script {

              if ( setGcloudProd )
              {
                //sh "gcloud config set account terraform-an@iceaccount.com"
                ENV_SELECT(true)
              }
              else
              {
                 sh "gcloud config set account jenkins@eaccount.com"
              }
                  sh 'docker images'
                      docker.image(DOCKER_IMAGE.id).push()
                      if ( !skipBUATSteps ){
                      docker.image(DOCKER_IMAGE_BUAT.id).push()
                      }
              }
          }
      }

      stage('Deploy in the cluster') {
          steps {
              script {
      if ( setGcloudProd )
              {
                //sh "gcloud config set account terraform-automation@comet-prod-c4c7.iam.gserviceaccount.com"
                ENV_SELECT(true)
        sh "kubectl --kubeconfig ${ENV}/cluster-config-file apply -f ${ENV}/${MS_NAME}.yaml"
              }
      else
              {
                 sh "gcloud config set account jenkins@comet-nonprod-200617.iam.gserviceaccount.com"
         sh "kubectl --kubeconfig ${ENV}/cluster-config-file apply -f ${ENV}/${MS_NAME}.yaml"
              }
      }
          }
      }

      stage('Remove Unused docker image') {
          when {
              expression {
                  !skipRemainingStages
              }
          }
          steps{
            script {
              sh "echo DOCKER_IMAGE.id"
              sh "docker rmi ${DOCKER_IMAGE.id}"
              if ( !skipBUATSteps){
                  sh "docker rmi ${DOCKER_IMAGE_BUAT.id}"
              }
              sh "docker images"
          }

          }
      }

      stage('CleanWorkspace post') {
          steps {
              step([$class: 'WsCleanup'])
              checkout scm
                      sh "ls -ltr"


          }
      }






  }
}
