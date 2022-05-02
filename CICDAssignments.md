# Ejercicios
 
## Ejercios Jenkins
 
### 1. CI/CD de una Java + Gradle
 
En el directorio raíz de este [código fuente](./jenkins-resources), crea un `Jenkinsfile` que contenga un pipeline declarativa con los siguientes stages:
 
* **Checkout** descarga de código desde un repositorio remoto, preferentemente utiliza GitHub.
* **Compile** compilar el código fuente, para ello utilizar `gradlew compileJava`
* **Unit Tests** ejecutar los test unitarios, para ello utilizar `gradlew test`
 
Para ejecutar Jenkins en local y tener las dependencias necesarias disponibles podemos contruir una imagen a partir de [este Dockerfile](./jenkins-resources/gradle.Dockerfile)
 
### 2. Modificar la pipeline para que utilice la imagen Docker de Gradle como build runner

* Utilizar Docker in Docker a la hora de levantar Jenkins para realizar este ejercicio.
* Como plugins deben estar instalados `Docker` y `Docker Pipeline`
* Usar la imagen de Docker `gradle:6.6.1-jre14-openj9`
 

---

### 1. Para crear la pipeline, lo he hecho tanto de forma escrita como usando la UI clásica, para la UI clásica no considero oportuno reflejar el proceso pues está genialmente detallado en: https://www.jenkins.io/doc/book/pipeline/getting-started/#through-the-classic-ui. Por otro lado, no sabía cómo implementar el checkout stage y leí en: https://www.jenkins.io/doc/pipeline/steps/workflow-scm-step/#checkout-check-out-from-version-control es más cómodo usar el approach del SCM plugin.

### Pipeline:

```
pipeline {
  agent any

  stages {
	
   stage('Checkout code') {
      steps {
        echo "Checking out project"
        // GIT checkout
        checkout scm: [
                $class: 'GitSCM',
                branches: [[name: '*/main']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [[$class: 'CleanCheckout']], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/M8anu/DevOps-Bootcamp-CICD-Exercises']]
        ]
      }
    }
    stage('Compile') {
      steps {    
        dir('./calculator'){
            echo "Compiling project"
            sh '''
            chmod +x gradlew
            ./gradlew compileJava
            '''
        }   
      }
    }
    stage('Unit Tests') {
      steps {
        dir('./calculator'){
          echo "Running unit tests" 
          sh './gradlew test'
        }
      }
    }
  }
}
```

### 2. Para crear la pipeline añadiendo la imagen de de Gradle Docker como build runner, lo he hecho tanto de forma escrita como usando la UI clásica, para la UI clásica no considero oportuno reflejar el proceso pues está genialmente detallado en: https://www.jenkins.io/doc/book/pipeline/getting-started/#through-the-classic-ui. Por otro lado, no sabía cómo implementar el checkout stage y leí en: https://www.jenkins.io/doc/pipeline/steps/workflow-scm-step/#checkout-check-out-from-version-control es más cómodo usar el approach del SCM plugin.

### Pipeline, para integrar docker, me he guiado por https://www.jenkins.io/doc/book/pipeline/docker/ :

```
pipeline {
  agent {
    docker {
      image 'gradle:6.6.1-jre14-openj9'
    }
  }

  stages {
    
   stage('Checkout code') {
      steps {
        echo "Checking out project"
        // GIT checkout
        checkout scm: [
                $class: 'GitSCM',
                branches: [[name: '*/main']], 
                doGenerateSubmoduleConfigurations: false, 
                extensions: [[$class: 'CleanCheckout']], 
                submoduleCfg: [], 
                userRemoteConfigs: [[url: 'https://github.com/M8anu/DevOps-Bootcamp-CICD-Exercises']]
        ]
      }
    }
    stage('Compile') {
      steps {
        dir("$WORKSPACE/calculator"){
            echo "Compiling project"
            sh '''
            chmod +x gradlew
            ./gradlew compileJava
            '''
        }    
      }
    }
    stage('Unit Tests') {
      steps {
        dir("$WORKSPACE/calculator"){
          echo "Running unit tests" 
          sh './gradlew test'
        }
      }
    }
  } 
}
```