pipeline {
 agent {
 node {
  label 'maven'
 }
 }

environment {
 RHT_OCP4_DEV_USER = 'ngqxba'
 DEPLOYMENT_STAGE = 'stage'
 DEPLOYMENT_PRODUCTION = 'production'
 }

 stages {
 stage('Tests') {
 steps {
 sh 'mvn clean install'
 }
 }
 stage('Package') {
 steps {
 sh '''
 mvn package -DskipTests \
 -Dquarkus.package.type=uber-jar
 '''
 archiveArtifacts 'target/*.jar'
 }
 }

  stage('Build Image') {
 environment { QUAY = credentials('QUAY_USER') }
 steps {
 sh '''
 mvn quarkus:add-extension \
 -Dextensions="kubernetes,container-image-jib"
 '''
 sh '''
  mvn package -DskipTests \
 -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
 -Dquarkus.container-image.build=true \
 -Dquarkus.container-image.registry=quay.io \
 -Dquarkus.container-image.group=$QUAY_USR \
 -Dquarkus.container-image.name=do400-deploying-environments \
 -Dquarkus.container-image.username=$QUAY_USR \
 -Dquarkus.container-image.password="$QUAY_PSW" \
 -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
 -Dquarkus.container-image.push=true
 '''
 }
}

stage('Deploy - Stage') {
 environment {
 APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-stage"
 QUAY = credentials('QUAY_USER')
 }
 steps {
 sh """
 cat <<EOF | oc apply -f -
 apiVersion: apps/v1
 kind: Deployment
 metadata:
   name: ${DEPLOYMENT_STAGE}
   namespace: ${APP_NAMESPACE}
 spec:
   replicas: 1
   selector:
     matchLabels:
       app: ${DEPLOYMENT_STAGE}
 template:
   metadata:
     labels:
       app: ${DEPLOYMENT_STAGE}
   spec:
     containers:
     - name: ${DEPLOYMENT_STAGE}
       image: quay.io/${QUAY_USR}/do400-deploying-environments:build-${BUILD_NUMBER}
 EOF
 """
 }
}

stage('Deploy - Production') {
 environment {
 APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-production"
 QUAY = credentials('QUAY_USER')
 }
 input { message 'Deploy to production?' }
 steps {
 sh """
 oc set image \
 deployment ${DEPLOYMENT_PRODUCTION} \
 shopping-cart-production=quay.io/${QUAY_USR}/do400-deploying-environments:build-${BUILD_NUMBER} -n ${APP_NAMESPACE} --record
"""
}
}

 }
}
