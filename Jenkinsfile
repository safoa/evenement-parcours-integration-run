#!groovy
import java.text.SimpleDateFormat

// pod utilisé pour la compilation du projet
podTemplate(label: 'evenement-parcours-integration-kubectl-pod', nodeSelector: 'medium', containers: [

        // le slave jenkins
        containerTemplate(name: 'jnlp', image: 'jenkinsci/jnlp-slave:alpine'),

        containerTemplate(name: 'envsubst', image: 'kristofferahl/envsubst', command: 'cat', ttyEnabled: true),

        // un conteneur pour déployer les services kubernetes
        containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl', command: 'cat', ttyEnabled: true)],

        // montage nécessaire pour que le conteneur docker fonction (Docker In Docker)
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node('gestion-images-kubectl-pod') {

        properties([
            parameters([string(defaultValue: 'DEFAULT', description: 'version à déployer', name: 'image')]),
            buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '',
                daysToKeepStr: '1', numToKeepStr: '3')),
            pipelineTriggers([])])

        stage('checkout sources') {
            checkout scm;
         }


        container('envsubst') {

            stage('parameterize') {
                withCredentials([string(credentialsId: 'registry_url', variable: 'registry_url')]) {

                    sh "IMAGE=${params.image} REGISTRY_URL=${registry_url} envsubst < deployment.yml > deployment-final.yml"
                }
            }
        }
        container('kubectl') {

            stage('deploy') {

                sh 'kubectl apply -f deployment-final.yml || :'
                sh 'kubectl apply -f ingress.yml || :'
            }
        }
    }
}
