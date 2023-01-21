pipeline {
    agent any
    environment {
        RT_SRV =        'ozamiro.jfrog.io'
        RT_USER =       credentials('rtuser')
        RT_PASS =       credentials('rtpass')
        RT_TRG_REPO =   'default-docker'
        IMG_NAME =      'ohadpetclinic'
    }
    stages {
        stage('Compile') {
            steps {
                echo 'Compiling..'
                sh './mvnw -U -Dmaven.test.skip=true clean compile'
                echo 'Done compiling'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
                sh './mvnw test'
                echo 'Test are complete. looks good.'
            }
        }    
        stage('Package') {
            steps {
                echo 'Building a Docker Image..'
                sh 'docker build . -t ""${IMG_NAME}""'
                echo 'Docker Image is ready!'
            }
        }
        stage('Publish') {
            steps {
                echo 'Publishing....'
                sh 'docker tag "${IMG_NAME}" "${RT_SRV}"/"${RT_TRG_REPO}"/"${IMG_NAME}"'
                sh 'docker login -u "${RT_USER}" -p "${RT_PASS}" "${RT_SRV}"'
                sh 'docker push "${RT_SRV}"/"${RT_TRG_REPO}"/"${IMG_NAME}"'
                echo 'The image is published.'
            }
        }
        stage('Cleanup') {
            steps {
                echo 'Cleaning up..'
                sh 'docker image prune -a'
            }
        }            
    }
}
