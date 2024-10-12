# Solución Semanal del Proyecto con GitHub Actions

Este documento describe paso a paso el enfoque adoptado para implementar un flujo de trabajo CI/CD utilizando GitHub Actions, Docker, y Nginx. A continuación, se detallan los componentes clave, problemas encontrados, y soluciones implementadas.

## 1. Diagrama del Flujo de Trabajo

Antes de comenzar la implementación, se creó un esquema básico del flujo de trabajo para obtener una visión clara de los pasos necesarios. Este esquema sirvió como guía para el desarrollo de los *workflows* en GitHub Actions.

## 2. Creación de Entornos y Secrets

Se configuraron los *environments* con sus correspondientes *secrets* en GitHub. Los entornos definidos incluyen **development** y **production**, cada uno con las variables `USERNAME` y `PASSWORD` necesarias para el login en Docker Hub.

![Environments Configuration](https://github.com/user-attachments/assets/e788b523-1eb2-47ae-ad59-2a0eda90a3ff)

## 3. Desarrollo del Workflow CI (`CI.yml`)

El siguiente paso fue desarrollar el flujo de trabajo de integración continua (CI). Se crearon tareas para automatizar el *build* y el *push* de la imagen Docker a Docker Hub.

### Flujo de Trabajo CI:

```yaml
name: CI
on:  
  workflow_call:
    outputs: 
      image_name: 
        value: ${{ jobs.ci.outputs.image_name }}

jobs: 
  ci_test:
    environment: production
    runs-on: ubuntu-latest
    steps:
      - name: Test 10 seconds
        run: |
          echo "EJECUTANDO TEST"
          sleep 10

      - name: Echo image_name
        run: |
          echo "FUNCIONA TODO PASAMOS A CD prueba"

  ci:
    needs: ci_test
    runs-on: ubuntu-latest
    environment: development
    outputs:
       image_name: ${{ steps.tag.outputs.image_name }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Checkout code again
        uses: actions/checkout@v3

      - name: Coger variables de package.json
        run: |
          VERSION=$(node -p "require('./package.json').version")
          NAME=$(node -p "require('./package.json').name")
          echo "VERSION=$VERSION" >> $GITHUB_ENV
          echo "NAME=$NAME" >> $GITHUB_ENV

      - name: crear tag
        id: create_tag
        uses: ./.github/actions/actiondockerpush
        with:
          image_name: '${{ env.NAME }}'
          version: '${{ env.VERSION }}'
          docker_username: ${{ secrets.USERNAME }}

      - name: Echo full image tags from action
        run: echo "${{ steps.create_tag.outputs.full_image_tags }}"

      - name: Create output variable
        id: tag
        run: echo "image_name=${{ steps.create_tag.outputs.full_image_tags }}" >> $GITHUB_OUTPUT 

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.USERNAME }}
          password: ${{ secrets.PASSWORD }}

      - name: Build Docker image
        run: docker build -t ${{ steps.create_tag.outputs.full_image_tags }} .

      - name: Push Docker image to DockerHub
        run: docker push ${{ steps.create_tag.outputs.full_image_tags }}
      
      - name: Echo output
        run: echo ${{ steps.tag.outputs.image_name }}
````
# Problemas Identificados en el CI

Durante el proceso de desarrollo, se encontró que el puerto en el archivo de configuración de Nginx se había cambiado del puerto por defecto 80 a 8080, lo que provocaba que el curl no funcionara correctamente. Este problema fue corregido modificando el comando `docker run` para mapear el puerto 8080 al puerto 80.

## Flujo de Trabajo CD (CD.yml)

Se desarrolló el flujo de trabajo de entrega continua (CD) para desplegar la aplicación desde Docker Hub y ejecutar el contenedor. Tamién añadiendo los aprobadores por entorno

### Flujo de Trabajo CD:

```yaml
name: CD
on:
  push:
    branches:
      - main
    paths:
      - '**'

workflow_call:
  inputs:
    image_name:
      required: true
      type: string
      default: 'vsanchezr33/sample-angular-app:0.0.1'

jobs:
  pull-DockerFiles:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.USERNAME }}
          password: ${{ secrets.PASSWORD }}
      - name: Pull Dockerfile
        run: |
          docker pull ${{ inputs.image_name }}
      - name: Run Docker container
        run: |
          docker run -d --name angular_aplication -p 80:8080 ${{ inputs.image_name }}

      - name: Wait 30 seconds for container to start 
        run: sleep 30

      - name: Docker ps
        run: |
          docker ps

      - name: Curl the deployed application
        run: |
          curl -I http://localhost:80
````
![image](https://github.com/user-attachments/assets/2870deae-9085-4277-ac9d-c90c04076532)

# Flujo de Trabajo Principal (principal.yml)
Finalmente, se creó un flujo de trabajo principal para integrar los workflows CI y CD, asegurando que los outputs del CI se pasen correctamente al CD.
## Flujo de Trabajo Principal:
````yml
name: semanal1
on:
  workflow_dispatch:
jobs:

  CI:
    if: github.ref == 'refs/heads/development'
    uses: ./.github/workflows/CI.yaml
    secrets: inherit

  CD:
    needs: CI
    uses: ./.github/workflows/CD.yml
    with:
      image_name: ${{ needs.CI.outputs.image_name }}
    secrets: inherit

  Echo_Output:
    needs: CI
    runs-on: ubuntu-latest
    steps:
      - name: Echo image_name
        run: echo "${{ needs.CI.outputs.image_name }}"
````
# Action Customizada para Crear la Tag
Se creó una GitHub Action personalizada para gestionar el push de la imagen Docker a Docker Hub. La acción genera las etiquetas completas de las imágenes y las exporta como outputs.
## Action:
````yml
# .github/actions/actiondockerpush/action.yml
name: Docker Push Action
description: "Push Docker image to DockerHub"
inputs:
  image_name:
    description: "The Docker image name"
    required: true
  version:
    description: "Version tag for the Docker image"
    required: true
  docker_username:
    description: "DockerHub username"
    required: true
outputs:
  full_image_tags:
    description: "Full image tags"
    value: ${{ steps.generate_full_image_tags.outputs.full_image_tags }}
runs:
  using: "composite"
  steps:
    - name: Generate full image tags
      id: generate_full_image_tags
      run: |
        full_tags="${{ inputs.docker_username }}/${{ inputs.image_name }}:${{ inputs.version }}"
        echo "full_image_tags=$full_tags" >> $GITHUB_OUTPUT
          
      shell: bash
````
# Solución a los Problemas con Variables

Un problema recurrente que surgió fue que la variable de la imagen image_name no se pasaba correctamente entre los workflows CI y CD. A pesar de probar distintos métodos, el problema persistió. 

Se tomaron las siguientes medidas para intentar resolverlo:
- Revisión del flujo de trabajo principal para asegurarse de que los outputs se pasaran correctamente desde el CI al CD.
- Comparación del código con ejemplos de compañeros y otras soluciones.
  
En este punto, el flujo completo está configurado para que, cuando la variable se pase correctamente, el proceso funcione de extremo a extremo.

## Conclusión

La verdad es que estoy bastante frustrado por el hecho de que la variable `image_name` no se haya pasado correctamente entre los workflows de CI y CD, a pesar de todos los intentos que hice para solucionarlo. Probé varias opciones, revisé ejemplos de otros compañeros y ajusté el código de múltiples formas, pero el problema persiste. Es desesperante cuando algo tan simple como pasar una variable se convierte en un obstáculo tan grande. 

Dicho esto, debo reconocer que este proyecto me ha enseñado muchísimo. He mejorado no solo mis habilidades técnicas, sino también mi capacidad de trabajar en equipo. La colaboración con mis compañeros y la búsqueda conjunta de soluciones me ha permitido entender mejor la importancia de la comunicación y la cooperación en un entorno de desarrollo. 

En resumen, aunque ha sido frustrante no poder resolver completamente el tema de las variables, me siento satisfecho con todo lo que he aprendido y cómo esto ha contribuido a mi crecimiento profesional. Si necesitas más detalles o quieres discutir alguna parte del proyecto, no dudes en contactarme. Estoy seguro de que juntos podemos encontrar una solución.

