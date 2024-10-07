# Ejercicios de Github Actions  para iniciarse en el Mundo de DevOps

¡Bienvenido/a al ejercicio semanal de Github Actions para iniciarse en el mundo de DevOps!

## Descripción

Este ejercicio tiene como objetivo introducirte al uso práctico de GitHub Actions en un entorno de CI/CD. CI/CD es un conjunto de prácticas que son fundamentales en el desarrollo de software moderno, especialmente en DevOps. Estas prácticas permiten una mayor agilidad, colaboración y calidad en el ciclo de vida del software, facilitando la entrega continua de nuevas funcionalidades y mejoras sin interrupciones importantes.

En un entorno DevOps, CI/CD se convierte en el puente que une el desarrollo y las operaciones, proporcionando a los equipos una plataforma donde pueden colaborar de manera fluida y eficiente. La Integración Continua (CI) garantiza que los desarrolladores puedan integrar su código de manera frecuente y segura en un repositorio central, donde se validan automáticamente los cambios mediante pruebas y builds. Esto ayuda a detectar errores lo antes posible y a mantener la calidad del código a medida que el proyecto crece.

Por otro lado, el Despliegue Continuo (CD) permite automatizar el proceso de entrega de software, asegurando que cada versión que pase las pruebas pueda ser desplegada de manera rápida y sin intervención manual. Esto reduce el tiempo de comercialización y mejora la fiabilidad del software, lo que es esencial en proyectos que necesitan ser escalables, seguros y robustos.

Durante este ejercicio, trabajarás con un proyecto básico de Angular, donde crearás los workflows necesarios para realizar el build, test y despliegue de la aplicación, integrando estos procesos automatizados en un pipeline de CI/CD.

## Objetivos

El propósito principal de este ejercicio es _proporcionarte una introducción práctica a los conceptos de Github Actions_ que son esenciales para cualquier persona interesada en trabajar en el área de DevOps. Al completar estos ejercicios, esperamos que adquieras experiencia práctica con:

- Creación de **workflows** en GitHub Actions.
- Uso de **Actions** personalizadas.
- Trabajo con **workflows reutilizables**.
- Manejo de **Secrets** y variables de entorno.
- Comprensión de los entornos de desarrollo y producción.

## Estructura del Repositorio

Este repositorio contiene una aplicación de Angular. Deberás diseñar dos workflows clave:
1. **Workflow de Integración Continua (CI)**: Se encargará de construir la aplicación, crear una imagen Docker y subirla a un registry (DockerHub).
2. **Workflow de Despliegue Continuo (CD)**: Tomará la imagen del registry, la desplegará y verificará que esté corriendo correctamente simulando una llamada a través de `curl` a la página HTML servida por **nginx**.

### Detalles del Despliegue
- **Rama `main`**: Será usada para los despliegues en el entorno de **producción**.
- **Rama `development`**: Se utilizará para los despliegues e integraciones continuas en el entorno de **UAT** (User Acceptance Testing).

### Requisitos Técnicos
Asegúrate de cumplir con los siguientes requerimientos:

1. **Triggers**: Configura correctamente los triggers, de forma que los workflows se ejecuten automáticamente en las condiciones necesarias.
   
2. **Workflows reutilizables**: Debes investigar y utilizar al menos un workflow reutilizable para no repetir código innecesario.

3. **Custom Actions**: Crea Actions personalizadas utilizando el tipo `composite` para una tareas específicas, como taggear la imagen antes de subirla a DockerHub, tal como se explica a continuación poseriormente.

4. **Variables y Secrets**:
   - Usa **variables de entorno** y **secrets** para gestionar información sensible como credenciales del Docker registry.
   - Diferencia entre entornos de **producción** y **uat** utilizando estas variables.

5. **Tests y cobertura de código**:
   - Debes crear un **job** para simular la ejecución de tests de cobertura de código. Este job debe ejecutarse solo cuando el entorno es de **producción**.
   - Si el entorno es **uat**, los tests no deben ejecutarse.

6. **Aprobadores por entorno**:
   - Investiga cómo configurar aprobadores para diferentes entornos.

### Custom Actions: Taggear la Imagen Docker

Uno de los objetivos del ejercicio es que crees una custom action de tipo composite para etiquetar (taggear) la imagen Docker antes de subirla al registry de DockerHub. La versión de la imagen será extraída dinámicamente del archivo package.json que se encuentra en el proyecto Angular del repositorio.

#### Especificaciones de la Custom Action:

1. **Entradas**:
    - `image_name`: El nombre de la imagen Docker (por ejemplo, `mi-app-angular`).
    - `version`: La versión de la imagen, que será extraída del archivo `package.json` del proyecto Angular. Debes leer el valor del campo "version" en este archivo y utilizarlo para etiquetar la imagen.

2. **Salida**:
    - La custom action debe devolver el nombre completo de la imagen etiquetada.

#### Implementación:

- Utilizarás esta custom action que vas a crear para obtener la etiqueta de la imagen en tu **workflow de CI**, justo antes de hacerl el build y subir la imagen a DockerHub.

#### Aclaraciones

Es muy importante que actualices manualmente la versión en el archivo package.json cada vez que quieras generar una nueva imagen Docker.

El campo "version" del archivo package.json es el que se usará para etiquetar la imagen Docker. Si no actualizas este campo antes de crear una nueva imagen, es posible que la imagen anterior sea sobreescrita (pisada) en el registry de DockerHub. Para evitar esto, debes:

- Abrir el archivo package.json en el proyecto Angular.
- Actualizar el valor del campo "version" (por ejemplo, de "1.0.0" a "1.0.1", "1.1.0", etc.).

## Contribución

¡Tus contribuciones son bienvenidas! Si tienes ideas para nuevos ejercicios o mejoras para los existentes, no dudes en abrir un issue o abrir un pull request.


