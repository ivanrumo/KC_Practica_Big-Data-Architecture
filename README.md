# KC_Practica_Big-Data-Architecture



## Objetivo

Para la realización de la práctica, me he puesto en el papel de una empresa de recruiting. Esta empresa necesita tener siempre una cartera de desarrolladores para poder ofrecer a sus clientes. Pero tienen el problema de que hay más oferta de trabajo de empresas que demanda de trabajo por parte de desarrolladores. Así que empiezan a desarrollar algunas ideas para encontrar talento de forma más rápida.

Una de las ideas que proponen es buscar a los técnicos que necesitan el StackOverflow.com. Es esta página los realizan preguntas relacionadas con el mundo del desarrollo y otros usuarios intentan ayudar a resolver las dudas o problemas planteados. Esta página cuenta con una API de la que se puede obtener mucha información de la página. Con la ayuda de este API, obtendrán información de los usuarios más activos a la hora de responder las dudas de otros usuarios. Así podrán contactar con estos usuarios para intentar incluirlos en su cartera de desarrolladores.

## Spring 1

Para obtener y clasificar la información necesaria se va usar el [API de Stack Exchange](https://api.stackexchange.com/). Con este API se puede obtener bastante información. Uno de los endpoints nos permite obtener las respuestas realizadas desde una determinada fecha. Utilizando este endpoint obtenemos los ids de los usuarios que han contestado alguna pregunta en los últimos días y los guardamos en un fichero. Si un usuario contesta varias veces durante el periodo de tiempo consultado, se escribirá varias veces en el fichero su id de usuario.

El API posee otro endpoint para obtener información de los usuarios a partir de su id. Con los ids de los usuarios obtenidos anteriormente, obtendremos los siguientes datos de los usuarios: Nombre, localización y reputación.

El primer fichero generado, lo usaremos como datos de entrada para el proceso WordCount. Así tendremos los usuarios junto con el número respuestas que han realizado. Este fichero y el de los datos de los usuarios, lo usaremos en Hive para generar consultas y obtener los datos de los usuarios más activos y las localizaciones más activas.

En el siguiente diagrama podemos ver la arquitectura que se va a necesitar para realizar estas tareas:



## Spring 2

Este spring y los siguientes lo he realizado en un [notebook de Google Colaboratory](https://colab.research.google.com/drive/1J3siKvUHW5e8HYg0GSSyqWjHWe0UBu7-). En el notebook se va explicando cuando se realiza cada spring:

* Primero se realizan los imports y se inicializan las variables con los valores adecuados: proyecto de Google cloud, nombre del cluster, zona, región, nombre del bucket de Google storage y nombre del fichero de credenciales de Google Cloud. Al ejecutar esta parte será necesariodar pemisos para que Google cloud pueda acceder a tu cuenta de Google y también solicitará que se proporcione un fichero de credenciales. El nombre del fichero que se suba tendrá que ser el mismo que haya indicado en la variable credentials_file.
* En el segundo paso se realizan consultas al API de StackExchange para obtener las respuestas que se ha realizado en stackoverflow.com en el último día. Se podrían consultar más días, pero el uso del API está limitado a 300 llamadas cada 6 horas, así que hay riesgo de superar la cuota si se consultan muchos días. Se realizan las consultas a esta url: https://api.stackexchange.com/2.2/answers. Cada petición devuelve 100 resultados y si hay mas resultados hay que volver ha hacer una consulta. De cada respuesta en la página obtenemos el id de usuario que ha realizado la respuesta y lo guardamos en una línea de un fichero que se denomina user_ids_answers. Al final de la ejecución se copia este fichero al Storage de Google que hemos configurado.
  * En el tercer paso se lee cada id del fichero user_ids_answers. Cada 100 ids de usuario se realiza una llamada al endpoint https://api.stackexchange.com/2.2/users/ para obtener los datos de los usuarios que han realizado respuestas. De cada usuario obtenemos el nombre, la reputación y su localización. Con estos datos y el id de usuario generamos un fichero csv, llamado user_ids_names, con una línea por cada usuario. Cuando se han consultado los datos de todos los usuarios, copiamos el fichero user_ids_names al Storage de Google que hemos configurado.

## Spring 3

En este spring se crea un cluster en Google dataproc. Este paso se también se realiza [en el mismo notebook](https://colab.research.google.com/drive/1J3siKvUHW5e8HYg0GSSyqWjHWe0UBu7-). Con los datos que se han configurado en el primer paso y [con la ayuda del API que proporciona Google](https://developers.google.com/api-client-library/python/) desplegamos automáticamente un cluster de Hadoop con un master y dos slaves.  Dado que no necesitamos gran potencia de proceso he configurado el cluster con las instancias menos potentes y un espacio en disco de 30GB cada una.

## Spring 4​	

En este spring,, en primer lugar, esperamos a que se termine de desplegar el cluster en Google Dataproc. Una vez que se ha desplegado el cluster, le manda ejecutar un trabajo Hadoop:

1. Este trabajo ejecuta el jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar
2. Se le indica que como parámetro realice un WordCount. 
3. Como ruta del fichero de entrada se indica el fichero user_ids_answers que hemos copiado al bucket de Google Storage. 
4. Como ruta de salida se indica la carpeta "output" que se generá en el HDFS del cluster. 
5. Una vez lanzado el trabajo, se espera a que termine de ejecutarse.

## Spring 5

En este spring lanzamos una trabajo en Hive que realiza las siguientes acciones:

1. Creamos la tabla user_answers con el contenido del fichero generado en hdfs por el trabajo de WordCount
2. Creamos la tabla users con el contenido del fichero que tiene el id del usuario, nombre, reputación y localización.
3. Creamos la tabla externa users_most_actives que generará un archivo en el bucket de Storage con los usuarios más activos.
4. Cargamos la tabla users_most_actives con una consulta en la que obtenemos los usuarios, con su id, su nombre y el número de respuestas que han realizado.
5. Creamos la tabla externa locations_most_actives que generará un archivo en el bucket de Storage con las localizaciones con los usuarios más activos.
6. Cargamos la tabla locations_most_actives con una consulta en la que obtenemos las localizaciones y el número de respuestas que han realizado los usuarios de cada localización. 

Por último consultamos los resultados que han generado las consultas realizada en Hive. Para ello utilizamos el comando gsutil para copiar los ficheros del Bucket de Storage a Google Drive y los visualizamos con el comando cat.

Por último, se elimina el cluster ya que ya se han realizado las tareas necesarias y por lo tanto no necesitamos incurrir en más gastos.



 