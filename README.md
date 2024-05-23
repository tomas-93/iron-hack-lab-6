# iron-hack-lab-6

## 1. Revise la configuración de canalización de CI/CD simulada:

<br>
1: Crear runner basico de dos pasos build y deploy

```` yaml
stages:
  - build
  - deploy

# Define el trabajo para construir la imagen Docker
build:
  stage: build
  script:
    # Inicia Docker en modo background
    - docker info
    # Construye la imagen Docker
    - docker build -t myapp:latest .
  tags:
    - docker

# Define el trabajo para desplegar la imagen Docker
deploy:
  stage: deploy
  script:
    # Ejecuta el contenedor Docker
    - docker run -d --name myapp -p 5000:5000 myapp:latest
  tags:
    - docker
  only:
    - main
````

2: Configurar rammas en runner, se agregar la rama dev y qa


```` yaml

on:
  push:
    branches:
      - dev
      - qa
      
stages:
  - build
  - deploy

# Define el trabajo para construir la imagen Docker
build:
  stage: build
  script:
    # Inicia Docker en modo background
    - docker info
    # Construye la imagen Docker
    - docker build -t myapp:latest .
  tags:
    - docker

# Define el trabajo para desplegar la imagen Docker
deploy:
  stage: deploy
  script:
    # Ejecuta el contenedor Docker
    - docker run -d --name myapp -p 5000:5000 myapp:latest
  tags:
    - docker
  only:
    - dev
    - qa
````

Con esta ultima configuracion de agregar on.push.branches especificamos 
que el runner debe ejecutarce en cuanto le hagan pull al la rama.


3: Crear docket file basico, definimos que es una app de python. y definimos el comando de inicio

`````` dockerfile
# Usa una imagen base oficial de Python
FROM python:3.9

# Establece el directorio de trabajo en el contenedor
WORKDIR /app


# Instala las dependencias
RUN pip install


# Define el comando de inicio
CMD ["python", "app.py"]
``````

4: Para las pruebas unitarias es necesario agregar otro step al runner 

```` yaml
# Define el trabajo para ejecutar pruebas automatizadas
test:
  stage: test
  script:
    # Construye una imagen específica para pruebas
    - docker build -t myapp-test:latest .
    # Ejecuta un contenedor para pruebas
    - docker run --rm myapp-test:latest python -m unittest discover -s tests
  tags:
    - docker
````
5: registro de la imgen a un repositorio, se define un nevo paso para registrar la imagen
<br>
Las siguiente variebles son configuradas en git
<li>$DOCKERHUB_USERNAME</li>
<li>$DOCKERHUB_PASSWORD</li>

```` yaml

# Define el trabajo para enviar la imagen Docker a un repositorio
push:
  stage: deploy
  script:
    # Inicia Docker en modo background
    - docker info
    # Autentica en Docker Hub
    - echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
    # Etiqueta la imagen
    - docker tag myapp:latest $DOCKERHUB_USERNAME/myapp:latest
    # Empuja la imagen al repositorio
    - docker push $DOCKERHUB_USERNAME/myapp:latest
  tags:
    - docker
  only:
    - main
````

6: Despliegue con kubernates, recordar que las variables se configurar en los secrets de git

```` yaml
# Define el trabajo para desplegar la imagen Docker en Kubernetes
deploy:
  stage: deploy
  script:
    # Instala kubectl
    - curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x ./kubectl
    - mv ./kubectl /usr/local/bin/kubectl
    # Configura kubectl con el archivo kubeconfig
    - echo "$KUBECONFIG" > kubeconfig
    - export KUBECONFIG=$(pwd)/kubeconfig
    # Despliega la nueva imagen en Kubernetes
    - kubectl set image deployment/myapp myapp=$DOCKERHUB_USERNAME/myapp:latest
  tags:
    - docker
  only:
    - dev
    - qa
````


yaml completo

```` yaml
on:
  push:
    branches:
      - dev
      - qa
      
stages:
  - build
  - test
  - push
  - deploy

# Define el trabajo para construir la imagen Docker
build:
  stage: build
  script:
    # Inicia Docker en modo background
    - docker info
    # Construye la imagen Docker
    - docker build -t myapp:latest .
  tags:
    - docker

# Define el trabajo para ejecutar pruebas automatizadas
test:
  stage: test
  script:
    # Construye una imagen específica para pruebas
    - docker build -t myapp-test:latest .
    # Ejecuta un contenedor para pruebas
    - docker run --rm myapp-test:latest python -m unittest discover -s tests
  tags:
    - docker

# Define el trabajo para enviar la imagen Docker a un repositorio
push:
  stage: push
  script:
    # Inicia Docker en modo background
    - docker info
    # Autentica en Docker Hub
    - echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
    # Etiqueta la imagen
    - docker tag myapp:latest $DOCKERHUB_USERNAME/myapp:latest
    # Empuja la imagen al repositorio
    - docker push $DOCKERHUB_USERNAME/myapp:latest
  tags:
    - docker
  only:
    - dev
    - qa

# Define el trabajo para desplegar la imagen Docker en Kubernetes
deploy:
  stage: deploy
  script:
    # Instala kubectl
    - curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x ./kubectl
    - mv ./kubectl /usr/local/bin/kubectl
    # Configura kubectl con el archivo kubeconfig
    - echo "$KUBECONFIG" > kubeconfig
    - export KUBECONFIG=$(pwd)/kubeconfig
    # Despliega la nueva imagen en Kubernetes
    - kubectl set image deployment/myapp myapp=$DOCKERHUB_USERNAME/myapp:latest
  tags:
    - docker
  only:
    - dev
    - qa


````

## 2. Analizar mejoras y posibles problemas:

Ventajas
<li> Escalado Horizontal</li>
<li> Despliegue Rápido y Eficiente </li>
<li> Entornos de Compilación Aislados </li>
<li> Consistencia en Diferentes Entornos </li>
<li> Pipeline Estable</li>

Desventajas
<li> Actualización y Mantenimiento </li>
<li> Rendimiento, Sobrecarga de Red </li>
<li> Errores de Configuración, problemas de red</li>

## 3. Escriba un informe de análisis:
#### Resume cómo se integra Docker en cada etapa del proceso de CI/CD.
<br>
Docker se puede integrar en cada etapa del proceso de CI/CD, desde la construcción de la imagen Docker, la ejecución de pruebas automatizadas, el registro de la imagen en un repositorio, hasta el despliegue de la imagen en Kubernetes. Docker proporciona un entorno aislado y consistente para ejecutar las diferentes etapas del proceso de CI/CD, lo que facilita la integración y la entrega continua de aplicaciones.
<br><br>

#### Analiza los beneficios y los posibles desafíos identificados durante la revisión.

Ventajas
<li> Escalado Horizontal</li>
<li> Despliegue Rápido y Eficiente </li>
<li> Entornos de Compilación Aislados </li>
<li> Consistencia en Diferentes Entornos </li>
<li> Pipeline Estable</li>

Desventajas
<li> Actualización y Mantenimiento </li>
<li> Rendimiento, Sobrecarga de Red </li>
<li> Errores de Configuración, problemas de red</li>
<br><br>


#### Sugiere soluciones teóricas o mejores prácticas para superar los desafíos.
<li> Actualización y Mantenimiento </li>

    Uso de softweare herramientas de escaneo para buscar vulnerabilidades en las imágenes de Docker

<li> Rendimiento, Sobrecarga de Red </li>

    Utiliza redes Docker eficientes y de alto rendimiento para minimizar la sobrecarga de red y mejorar el rendimiento de la comunicación entre contenedores.

