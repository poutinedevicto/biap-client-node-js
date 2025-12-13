pipeline {
  agent {
    // LOCAVORA_TODO agent also defined in other Jenkinsfile 
    // LOCAVORA - low idleMinutes because we ask for a lot of resources
    kubernetes {
      label 'jenkins-agent-kaniko-remote-harbor'
      idleMinutes 1 // Keep the Pod alive for n minutes after the build
      yaml '''
apiVersion: v1
kind: Pod
metadata:
  name: kaniko-builder
spec:
  volumes:
    - name: docker-config-secret
      secret:
        # Secret provient de Vault via un External Secret
        secretName: harbor-beckn-locavora-rw-external-secret
    # Volume pour stocker temporairement le contexte (code source) et l'image de sortie
    #- name: shared-data
    #  emptyDir: {}
  
  containers:  
  - name: kaniko
    # 'latest' aussi disponible mais 'debug' init. parce que sh dispo
    image: martizih/kaniko:debug
    # LOCAVORA - on execute sleep infinity to keep the container running
    # les steps "container('kaniko')" vont exec des commandes dans ce container
    command:
      - /busybox/sh
      - -c
      - "sleep infinity"
        
    volumeMounts:
      - name: docker-config-secret
        mountPath: /kaniko/.docker
        readOnly: true

    #resources:
    #  requests:
    #    memory: "2Gi"
    #    cpu:    "1"
    #    # LOCAVORA - max total ephemeral-storage per node under GKE 
    #    #            is 10Gi, and the other container (jnlp) requests 1Gi 
    #    ephemeral-storage: 9Gi

  restartPolicy: Never
'''   
    }
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    durabilityHint('PERFORMANCE_OPTIMIZED')
    disableConcurrentBuilds()
  }
  environment {
    REGISTRY_NAME = 'harbor.beckn.locavora.org'
    IMAGE_NAME =    'locavora-public/biap-client-node-js'
    CACHE_NAME =    'locavora/dockerfile-build-cache'
  }
  stages {

    // LOCAVORA - attention on doit utiliser les apos (') pour que les variables d'environnement ne soient pas interprétées
    stage('Build with Kaniko Dockerfile in provided repo') {
      steps {
        container('kaniko') {
          sh '/kaniko/executor -c . \
              --cache=true \
              --cache-repo $REGISTRY_NAME/$CACHE_NAME \
              --destination $REGISTRY_NAME/$IMAGE_NAME:latest'
        }
      }
    }
  }
}