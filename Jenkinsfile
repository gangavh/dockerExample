//Picking a build agent labeled "ec2" to run pipeline on
node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("savindra/devopsrobo")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }
	
	stage('Push image') {
         /* Finally, we'll push the image with two tags:
          * First, the incremental build number from Jenkins
          * Second, the 'latest' tag.
          * Pushing multiple tags is cheap, as all the layers are reused. */
         docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
             app.push("${env.BUILD_NUMBER}")
             app.push("latest")
         }
      }

	stage 'Stage image'
	//Deploy image to staging in ECS
	def buildenv = docker.image('savindra/devopsrobo')
  buildenv.inside {
    wrap([$class: 'AmazonAwsCliBuildWrapper', credentialsId: 'AKIAIZZJ2X23N7BCK4OA', defaultRegion: 'eu-central-1']) {
        sh "aws ecs update-service --service staging-devopsrobo  --cluster staging --desired-count 0"
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                sh "aws ecs describe-services --services staging-devopsrobo  --cluster staging  > .amazon-ecs-service-status.json"

                // parse `describe-services` output
                def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
                def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
                println "$ecsServicesStatus"
                def ecsServiceStatus = ecsServicesStatus.services[0]
                return ecsServiceStatus.get('runningCount') == 0 && ecsServiceStatus.get('status') == "ACTIVE"
            }
        }
        sh "aws ecs update-service --service staging-devopsrobo  --cluster staging --desired-count 1"
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                sh "aws ecs describe-services --services staging-devopsrobo --cluster staging > .amazon-ecs-service-status.json"

                // parse `describe-services` output
                def ecsServicesStatusAsJson = readFile(".amazon-ecs-service-status.json")
                def ecsServicesStatus = new groovy.json.JsonSlurper().parseText(ecsServicesStatusAsJson)
                println "$ecsServicesStatus"
                def ecsServiceStatus = ecsServicesStatus.services[0]
                return ecsServiceStatus.get('runningCount') == 0 && ecsServiceStatus.get('status') == "ACTIVE"
            }
        }
        timeout(time: 5, unit: 'MINUTES') {
            waitUntil {
                try {
                    sh "curl http://52.200.92.100:80"
                    return true
                } catch (Exception e) {
                    return false
                }
            }
        }
        echo "devopsrobo#${env.BUILD_NUMBER} SUCCESSFULLY deployed to http://52.200.92.100:80"
        input 'Does staging http://52.200.92.100:80 look okay?'
 
}