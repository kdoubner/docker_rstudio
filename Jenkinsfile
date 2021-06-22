def makeDockerImageVersion(){
  if(env.BRANCH_NAME == 'master'){
    return 'latest'
  }
  return env.BRANCH_NAME
}

pipeline{
  agent {
    label "analytics"
  }

  options {
    timestamps()
  }

  environment {
    registry = "657399224926.dkr.ecr.us-east-1.amazonaws.com/rstudio"
    registryCredential = 'ecr:us-east-1:ta_jenkins'
    GIT_CREDS = credentials('ddd652c3-3358-4ca5-8183-8bdd9024bdc0')
  }

  stages{
    stage("Prepare building environment"){
      steps{
        script{
          sh 'env'
          sh 'bash addons/render.sh'
          sh '''#!/bin/bash
          for i in dv rbase rstudio; do
            echo $i;
            cp addons/* $i/
            cp Packages_analytics.* $i/
            touch $i/Packages_dummy.py
          done;
          rm dv/Packages_analytics.*
          cp Packages_datavalidation.* dv/
          touch dv/Packages_dummy.py
          '''

          milestone()
        }
      }
    }
    stage("Build base images"){
      parallel{
        // stage("Datavalidation image"){
        //   steps{
        //     script{
        //       ansiColor('xterm') {
        //         dvImage = docker.build("${registry}:dv-${makeDockerImageVersion()}", "./dv")
        //       }
        //     }
        //   }
        // }

        stage("RStudio"){
          steps{
            script{
              ansiColor('xterm') {
                rstudioImage = docker.build("${registry}:${makeDockerImageVersion()}", "--build-arg git_creds=$GIT_CREDS ./rstudio")
              }
            }
          }
        }

        stage("Analytical team image"){
          steps{
            script{
              ansiColor('xterm') {
                rbaseImage = docker.build("${registry}:rbase-${makeDockerImageVersion()}", "--build-arg git_creds=$GIT_CREDS ./rbase")
              }
            }
          }
        }
      }
    }

    stage("Build development image"){
      steps{
        script{
          ansiColor('xterm') {
            sh "sed -r 's!%%CONTAINER_VERSION%%!${makeDockerImageVersion()}!g;' test/Dockerfile.template > test/Dockerfile"
            testImage = docker.build("${registry}:test-${makeDockerImageVersion()}", "--build-arg git_creds=$GIT_CREDS ./test")
          }
        }
      }
    }


//     stage("Publish to ECR"){
//       // Skip docker image publish when pull request
//       when{
//         not { branch 'PR-*' }
//       }
//       steps{
//         script{
//           docker.withRegistry('https://657399224926.dkr.ecr.us-east-1.amazonaws.com', registryCredential) {
//             // dvImage.push()
//             rstudioImage.push()
//             rbaseImage.push()
//             testImage.push()
//           }
//         }
//       }
//     }
  }
//   post{
//     always{
//       script{
//         deleteDir()
//       }
//     }
//     failure{
//       script {
//         emailext subject: "Build# ${env.BUILD_NUMBER} Docker image ${registry} failed",
//                    body: '${SCRIPT, template="groovy-html.template"}',
//                    mimeType: 'text/html',
//                    from: "jenkins@finmason.com",
//                    replyTo: "ops@finmason.com",
//                    recipientProviders: [
//                             [$class: 'CulpritsRecipientProvider'],
//                             [$class: 'DevelopersRecipientProvider'],
//                             [$class: 'RequesterRecipientProvider']]
//       }
//     }
//   }
}
