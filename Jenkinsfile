#!/usr/bin/env groovy

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>' //reading the new version value from pom.xml 
                    def version = matcher[0][1] //access the first child element and the first file in it (parsing xml files)
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER" //build number environment variable from jenkins
                } //notice also the way to handle variables inside the shell script \\\${parsedVersion.majorVersion}
            }
        }
        stage('build app') {
            steps {
                script {
                    echo "building the application..."
                    sh 'mvn clean package' //to delete the old jar file before creating the new file
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t ibrahimosama/my-repo:${IMAGE_NAME} ."
                        sh "echo \$PASS | docker login -u \$USER --password-stdin"
                        sh "docker push ibrahimosama/my-repo:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    echo 'Committing version update to Github'
                    withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        // git config here for the first time run
                        sh 'git config --global user.email "iosama.amin@gmail.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh "git remote set-url origin https://${PASS}@github.com/ibrahim-osama-amin/java-maven-app-with-versioning.git"
                        sh 'git add .'
                        sh 'git commit -m "CI: version bump"'
                        sh 'git push main'
                    }
                }
            }
        }
    }
}
