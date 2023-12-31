node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */
        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */
        app = docker.build('skandersoltane/flask-hello-world')
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-id') {
            app.push("${env.BUILD_NUMBER}")
            app.push('latest')
        }
    }

    stage('Ansible') {
        /* Move the Ansible playbook execution directly within the stage */
        script {
            sh 'ansible-playbook kubectl-setup-playbook.yaml'
        }
    }

    stage('Deploy') {
        script {
            sh """
                kubectl set image deployment/hello-world hello-world=skandersoltane/flask-hello-world:"${env.BUILD_NUMBER}"
            """
        }
    }
}
