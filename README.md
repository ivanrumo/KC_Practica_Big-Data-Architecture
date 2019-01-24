# KC_Practica_Big-Data-Architecture



## Objetivo

Para la realización de la práctica me he puesto en el papel de una empresa de recruiting. Esta empresa necesita tener siempre una cartera de desarrolladores para poder ofrecer a sus clientes. Pero tienen el problema de que hay más oferta de trabajo de empresas que demanda de trabajo por parte de desarrolladores. Así que empiezan a desarrollar algunas ideas para encontrar talento de forma más rápida.

Una de las ideas que proponen es buscar a los técnicos que necesitan el StackOverflow.com. Es esta página los realizan preguntas relacionadas con el mundo del desarrollo y otros usuarios intentan ayudar a resolver las dudas o problemas planteados. Esta página cuenta con una API de la que se puede obtener mucha información de la página. Con la ayuda de este API obtendrán información de los usuarios más activos a la hora de responder las dudas de otros usuarios. Así podrán contactar con estos usuarios para intentar incluirlos en su cartera de desarrolladores.

## Spring 1

Para obtener y clasificar la información necesaria se va usar el [API de Stack Exchange](https://api.stackexchange.com/). Con este API se puede obtener bastante información. Uno de los endpoints nos permite obtener las respuestas realizadas desde una determinada fecha. Utilizando este endpoint obtenemos los ids de los usuarios que han contestado alguna pregunta en los últimos días y los guardamos en un fichero. Si un usuario contesta varias veces, durante el periodo de tiempo consultado se escribirá varias veces en el fichero.

El API posee otro endpoint para obtener información de los usuarios a partir de su id. Con los ids de los usuarios obtenidos anteriormente obtendremos los datos de los usuarios: Nombre, localización y reputación.

El primer fichero lo usaremos como datos de entrada para el proceso WordCount. Así tendremos los usuarios con las respuestas que han respondido. Este fichero y el de los datos de los usurios, lo usaremos en Hive para generar consutas y obtener los datos de los usuarios más activos, las localizaciones más activas, etc.

En el siguiente diagrama podemos ver la arquitectura que se va a necesitar para realizar estas tareas:

## Spring 2

Este spring y los siguientes lo he realizado en un [notebook de Google Colaboratory](https://colab.research.google.com/drive/1J3siKvUHW5e8HYg0GSSyqWjHWe0UBu7-). 



 