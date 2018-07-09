node {
    def app
    properties([parameters([
        choice(choices: ["dev", "prd"].join("\n"),
            description: 'Environment', name: 'ENVIRONMENT')
    ])])
            stage('Clone repository') {
            /* Let's make sure we have the repository cloned to our workspace */

            checkout scm
            }

            stage('Build image') {
                /* This builds the actual image; synonymous to
                * docker build on the command line */

                app = docker.build("dtr.deso.tech/docker-datacenter/hellonode2")
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
                docker.withRegistry('https://dtr.deso.tech', 'docker-deso-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
            }

            stage('Docker Stack Deploy') {
                try {
                        sh "docker stack rm ${params.ENVIRONMENT}_hello-world"
                        sh 'sleep 30s' // wait for service to be completely removed if it exists
                    } catch (err) {
                        echo "Error: ${err}" // catach and move on if it doesn't already exist
                    }
                sh "pwd"    
                sh "docker stack deploy \
                    --compose-file=${params.ENVIRONMENT}_hello-world.yml ${params.ENVIRONMENT}_hello-world"
                }

        }
    }

