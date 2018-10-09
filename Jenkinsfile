#!groovy

node { /* This is required to distribute the work to nodes */
	def app
	
     stage('Clone Repo') {
	/* checkout the code to build */
	checkout scm
     }

        stage('build ONOS') {
                sh '''#!/bin/bash -l
                    ONOS_ROOT=`pwd`
                    source tools/build/envDefaults
                    onos-buck build onos
                '''
        }
	
     stage('Unit Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */
		 	parallel (
                    "unit-tests": {
                        sh '''#!/bin/bash -l
                            ONOS_ROOT=`pwd`
                            source tools/build/envDefaults
                            onos-buck test
                        '''
                    },
                    "javadocs": {
                        sh '''#!/bin/bash -l
                            ONOS_ROOT=`pwd`
                            source tools/build/envDefaults
                            onos-buck build //docs:external //docs:internal --show-output
                        '''
                    },
	     )
    }
	
    stage('Build docker image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("mohdjahangir/onos-dev")
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
}
