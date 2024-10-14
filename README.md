# Control de Autoría Github Actions

# Ejercicio de Modificación - Gestión de Etiquetas en Función de la Rama

¡Bienvenido/a al nuevo ejercicio! En esta ocasión, vamos a ampliar las funcionalidades del trabajo que realizaste en el ejercicio anterior. Ahora tendrás que modificarlo para que sea capaz de **detectar en qué rama se han producido los cambios** y ajustar las etiquetas de la imagen Docker en consecuencia.

## Descripción

El objetivo de este ejercicio es realizar una modificación en tu workflow de **GitHub Actions** para que maneje las etiquetas de las imágenes Docker según la rama en la que se hayan realizado los cambios:

- Si los cambios se han realizado en la **rama `main`**, la etiqueta de la imagen Docker no cambiará. (El valor de la etiqueta sigue estando formado por los campos del nombre y la versión que figuran en el fichero package.json)
- Si los cambios se han realizado en la **rama `development`**, debes añadir `-snapshot` a la etiqueta de la imagen Docker.

Este ejercicio te permitirá afianzar tu conocimiento en el manejo de ramas y condicionales dentro de los workflows de GitHub Actions.

## Objetivos

Al completar este ejercicio, deberás ser capaz de:

- Detectar la rama en la que se ha realizado el commit.
- Condicionar el etiquetado de una imagen Docker en función de la rama.
- Aplicar lógica dentro de un workflow de GitHub Actions.



## Aclaraciones

Una vez hayas actualizado la **custom action** para manejar el etiquetado de la imagen Docker en función de la rama, asegúrate de integrarla dentro de tu **workflow de CI** y de que tu **workflow de CD** utilice la imagen correspondiente.

Es muy importante que actualices manualmente la versión en el archivo package.json cada vez que quieras generar una nueva imagen Docker.
 
El campo "version" del archivo package.json es el que se usará para etiquetar la imagen Docker. Si no actualizas este campo antes de crear una nueva imagen, es posible que la imagen anterior sea sobreescrita (pisada) en el registry de DockerHub.