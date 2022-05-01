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
 
## Ejercicios GitLab
 
### 1. CI/CD de una aplicación spring
 
* Crea un nuevo proyecto en GitLab y un repositorio en el mismo, para la aplicación `springapp`. El código fuente de la misma lo puedes encontrar en este [enlace](../02-gitlab/springapp).
* Sube el código al repositorio recientemente creado en GitLab.
* Crea una pipeline con los siguientes stages:
 * maven:build - En este `stage` el código de la aplicación se compila con [maven](https://maven.apache.org/).
 * maven:test - En este `stage` ejecutamos los tests utilizando [maven](https://maven.apache.org/).
 * docker:build - En este `stage` generamos una nueva imagen de Docker a partir del Dockerfile suministrado en el raíz del proyecto.
 * deploy - En este `stage` utilizamos la imagen anteriormente creada, y la hacemos correr en nuestro local
 
* **Pistas**:
 - Utiliza la versión de maven 3.6.3
 - El comando para realizar una `build` con maven: `mvn clean package`
 - El comando para realizar los tests con maven: `mvn verify`
 - Cuando despleguemos la aplicación en local, podemos comprobar su ejecución en: `http://localhost:8080`
 
En resumen, la `pipeline` de `CI/CD`, debe hacer la build de la aplicación generando los ficheros jar, hacer los tests de maven y finalmente dockerizar la app (el dockerfile ya se proporciona en el repo) y hacer un deploy en local.
 
### 2. Crear un usuario nuevo y probar que no puede acceder al proyecto anteriormente creado
* Añadirlo con el role `guest`, comprobar que acciones puede hacer.
* Cambiar a role `reporter`, comprobar que acciones puede hacer.
* Cambiar a role `developer`, comprobar que acciones puede hacer.
* Cambiar a role `maintainer`, comprobar que acciones puede hacer.
 
* **Nota** (acciones a probar):
 - Commit
 - Ejecutar pipeline manualmente
 - Push and pull del repo
 - Merge request
 - Acceder a la administración del repo
 
### 3. Crear un nuevo repositorio, que contenga una pipeline, que clone otro proyecto, springapp anteriormente creado. Realizarlo de las siguientes maneras:
  
* Con el método de CI job permissions model
   - ¿Qué ocurre si el repo que estoy clonando no estoy cómo miembro?
 > Pista: https://docs.gitlab.com/ee/user/project/new_ci_build_permissions_model.html (Dependent Repositories)
 * Con el método deploy keys
   - Crear deploy key en el repo springapp y poner solo lectura
   - Crear pipeline que usando la deploy key
 > Pista: https://docs.gitlab.com/ee/ci/ssh_keys/
 

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