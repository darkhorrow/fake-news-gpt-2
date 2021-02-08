# Generación de noticias con GPT-2

## Introducción

En este proyecto se plantea generar noticias mediante el modelo GPT-2. Para ello, se realiza un *fine-tuning* de este modelo, de manera que el texto generado con GPT-2 tenga mayor similaridad con el conjunto de datos usados durante el re-entrenamiento.

En este caso concreto, se pretende genrar noticias relacionadas con la provincia de Las Palmas.

## GPT-2

GPT-2 es un modelo de lenguaje desarrollado por **OpenAI**.

Tal y como se titula el artículo científico acerca de este modelo, *'Language Models are Unsupervised Multitask Learners'*, el conjuntode entrenamiento utilizado con el modelo carece de etiquetas de clasificación ya que usa, a grandes rasgos, texto en sí mismo. El conjunto de datos se obtuvo principalmente mediante *Web scrapping*, más concretamente de lugares como **Common Crawl**.

El *paper* de GPT-2 es el siguiente: https://d4mucfpksywv.cloudfront.net/better-language-models/language-models.pdf

El texto que se usa como entrada para la arquitectura de 'Transformer' subyacente (tema que trataremos más adelante) se preprocesa, realizando una conversión a *Byte-Pair Encoding*. Extraído literalmente de *Wikipedia*, el Byte-Pair Enconding *'es una forma simple de compresión de datos en la que el par más común de bytes consecutivos de datos se reemplaza con un byte que no ocurre dentro de esos datos'*

Se puede encontrar todo el ejemplo de esta codificación en: https://es.wikipedia.org/wiki/Codificaci%C3%B3n_de_pares_de_bytes

GPT-2 está constituido por una serie de bloques de 'transformer' de decodificación (transformer decoder blocks, para ser más claro). Por ello, GPT-2 se caracteriza por producir resultados token por token.

Otra característica que hace muy potente a GPT-2 es lo que se denomina *auto-regresión*, que consiste en introducir como parte de los datos de entrada los token producidos en instantes anteriores.

Existen muchas más características en GPT-2 que se pueden mencionar, las cuáles se pueden encontrar en este psot, que lo ilustra de una manera muy decente: http://jalammar.github.io/illustrated-gpt2/

## Conjunto de datos usado

El primer detalle a comentar con respecto a los datos de entenamiento es que, debido al gran tamaño inicial que tenía (¡15GB!), se escogió, únicamente, las noticias procedente de la provincia de Las Palmas (los datos originales contenían todas las provincias de España). Sin embargo, no resultó ser una acotación suficiente para mi espacio de almacenamiento en Google Drive, por lo que tomé sólo unos 40MB de Las Palmas a su vez, motivo por el cuál pueden existir carpetas vacías.

Con esto aclarado, se procede a definir el formato de los datos.

Las noticias se encuentran en subcarpetas que representan una fecha concreta y están en formato JSON. Este fichero contiene los datos que ilustramos a continuación:

    {
        "province": "LAS PALMAS",
        "date": "2009-03-11T00:00:00",
        "url": "https://www.20minutos.es/noticia/456132/0/comida/supermercados/crisis/",
        "title": {
            "raw_text": "Canarias repartirá a las familias necesitadas la comida sobrante de los supermercados",
            "lemmatized_text": "canario repartir a el familia necesitar el comida sobrante de el supermercado",
            "lemmatized_text_reduced": "canario repartir familia necesitar comida sobrante supermercado",
            "persons": [],
            "locations": [],
            "organizations": [],
            "others": [],
            "dates": [],
            "numbers": []
        },
        "lead": {
            "raw_text": "Serán alimentos que hayan superado su fecha de consumo preferente. Se espera que los hipermercados se adhieran a esta iniciativa. ENCUESTA: ¿Tomarías productos cuya fecha de consumo preferente haya expirado? CONSULTA AQUÍ MÁS NOTICIAS DE LAS PALMAS .",
            "lemmatized_text": "ser alimento que haber superar su fecha de consumo preferente se esperar que el hipermercado se adherir a este iniciativa encuesta tomar producto cuyo fecha de consumo preferente haber expirar consulta_aquí_más_noticias_de_las_palmas",
            "lemmatized_text_reduced": "ser alimento haber superar fecha consumo preferente esperar hipermercado adherir iniciativa encuesta tomar producto fecha consumo preferente haber expirar consulta_aquí_más_noticias_de_las_palmas",
            "persons": [],
            "locations": [],
            "organizations": [
                "CONSULTA_AQUÍ_MÁS_NOTICIAS_DE_LAS_PALMAS"
            ],
            "others": [],
            "dates": [],
            "numbers": []
        },
        "body": {
            "raw_text": "El Gobierno insular ha acordado con la Asociación de Supermercados de Canarias (Asuican) repartir los alimentos sobrantes de los establecimientos entre las personas que más lo necesiten , el mismo día o el día después de que sean retirados de los expositores. Según el principio de acuerdo que han alcanzado Gobierno y empresas, se distribuirán entre las familias en situación de emergencia los alimentos que estén a punto de caducar y que los supermercados no vayan a comercializar. Los productos habrán superado su fecha de consumo preferente , pero no estarán caducados, según el portavoz del Gobierno canario, Martín Marrero, citado por la prensa local. Los criterios que determinarán el reparto serán establecidos por la Consejería de Bienestar Social . Desde el gabinete esperan que los hipermercados se adhieran a esta iniciativa, que pretende llegar a todas las islas. NOTICIAS DE LAS PALMAS",
            "lemmatized_text": "el gobierno insular haber acordar con el asociación_de_supermercados_de_canarias asuican repartir el alimento sobrante de el establecimiento entre el persona que más lo necesitar el mismo día o el día después_de que ser retirar de el expositor según el principio de acuerdo que haber alcanzar gobierno y empresa se distribuir entre el familia en situación de emergencia el alimento que estar a_punto_de caducar y que el supermercado no ir a comercializar el producto haber superar su fecha de consumo preferente pero no estar caducar según el portavoz de el gobierno canario martín_marrero citar por el prensa local el criterio que determinar el reparto ser establecer por el consejería_de_bienestar_social desde el gabinete esperar que el hipermercado se adherir a este iniciativa que pretender llegar a todo el isla noticias_de_las_palmas",
            "lemmatized_text_reduced": "gobierno insular haber acordar asociación_de_supermercados_de_canarias asuican repartir alimento sobrante establecimiento persona necesitar mismo día día ser retirar expositor principio acuerdo haber alcanzar gobierno empresa distribuir familia situación emergencia alimento estar caducar supermercado ir comercializar producto haber superar fecha consumo preferente estar caducar portavoz gobierno canario martín_marrero citar prensa local criterio determinar reparto ser establecer consejería_de_bienestar_social gabinete esperar hipermercado adherir iniciativa pretender llegar isla noticias_de_las_palmas",
            "persons": [
                "Martín_Marrero"
            ],
            "locations": [],
            "organizations": [
                "Gobierno",
                "Asociación_de_Supermercados_de_Canarias",
                "Asuican",
                "Gobierno",
                "Gobierno",
                "Consejería_de_Bienestar_Social"
            ],
            "others": [
                "NOTICIAS_DE_LAS_PALMAS"
            ],
            "dates": [],
            "numbers": []
        }
    }


La parte que nos interesa de los datos es el "body", que contiene el cuerpo principal de la noticia.

---

Nota: El formato de lso datos es bastante útil para el ámbito del NLP, ya que posee anotaciones de entidades y lematización incorporada.

---

## Desarrollo

Si bien el notebook de Jupyter generado posee una breve explicación de los pasos seguidos, en este apartado realzaremos algunas decisiones tomadas y posibles mejoras a realizar.

En primer lugar, con el objetivo de entrenar con un conjunto inicial en español y retornar los resultados en esàñol también, usaremos una librería que se comunica con Google Translate.

---
Nota: Es importante destacar el uso de google_trans_new en lugar de googletrans. Hasta la fecha actual [08/02/2021] hay un problema con googletrans que impide enviar solicitudes.

---


A continuación, cargamos el contenido de Google Drive. Eso lo realizamos, ya que los ficheros/directorios almacenados en el entorno del Google Colab se eliminan cuando se recicla dicho entorno, comportamient que evidentemente no se desea.

Con el Drive conectado, se crea un directorio y clona un repositorio, en este caso, un *fork* del repositorio original de OpenAI. Este repositorio cuenta con los ficheros train.py y conditional_model.py, que nos permitirán re-entrenar y obtener una muestra de GPT-2, respectivamente.

Tras esto, se crea un directorio más, en el que se debe tener el conjunto de datos LAS PALMAS. Como train.py espera que los textos se le pasen de manera unificada en un fichero .txt, realizamos la conversión. Aunque está limitado a solo las primeras 50 noticias que itere (son muchas a procesar y todas ellas llevan mucho tiempo), esto se podría quitar para usar los 40MB de noticias.

Una vez generado el fichero de texto plano único, lo traducimos línea a línea al inglés (GPT-2 es un modelo en inglés exclusivo). La librería google_trans_new tiene un límite de 5000 caracteres por petición, por ello, debemos iterar.

A continuación, se instalan las dependencias de GPT-2 y se resuelve una serie de incompatibilidades de tensorflow-gpu como CUDA con las versiones existentes en Google Colab.

Los últimos pasos consisten en asignar los permisos adecuados a todos los ficheros/directorios bajo nuestro proyecto para poder ejecutar el entrenamiento y... ¡entenar!

---
Nota: Recuerda poner el entorno de ejecución de Google Colab a GPU. El entrenamiento con CPU es muy lento y no funciona con TPU, según las pruebas realizadas.

---

El entrenamiento tiene la peculiaridad de no tener fin, se puede dejar el tiempo que se desee e interrumpir la ejecución para detenerlo. Durante la ejecución, se crea un *checkpoint* de cata iteración y otro aparte cada 1000. Hay que tener cuidado con esto, ya que los modelos supera el espacio de almacenamiento de 1GB y podría llenar el Drive si no se controla.

Cuando se pare el entrenamiento, se procede a usar el conditional_model para obtener una muestra a partir de una frase de entrada, mediante el campo de texto presente en el notebook.

Traduciendo tanto el texto de entrada como el de salida, obtenemos los resultados. Teniendo en cuenta que existe un proceso estocástico de fondo, si se quiere obtener el mismo resultado bajo la misma entrada, podemos pasar el parámetro *seed* al conditinal_model.

Como ejemplo para la frase de entrada *'Santa Brígida es'*, se obtiene la siguiente noticia:

*'Santa Brígida es un pequeño pueblo costero de 30 casas. Se encuentra en la confluencia de Côte d'Ivoire y el Océano Atlántico, y fue fundada como un puesto comercial de España en el siglo XIII, con la intención de ser un puente entre Europa y África. Solo en el siglo XIX fue reconstruido gracias a los esfuerzos de científicos, misioneros y aventureros europeos. Pero este es quizás el único pueblo que sobrevive en una región de unos pocos miles de habitantes (según mi estimación) que algunos consideran 'habitable', al menos en esta medida, según el antropólogo John Morton: "Se puede estimar en entre 6.000 y 5.000 habitantes. El pueblo de Brígida es casi tan numeroso como el que existía durante la época de su fundación, aunque no está completamente rodeado por el Atlántico ”. La zona había estado habitada durante algún tiempo, pero solo recientemente surgió la idea de establecer "una nueva comunidad de asentamiento y comercio entre la gente del Congo en un lugar donde no se encuentra la gente de todas las demás tribus del Congo". El pueblo y sus alrededores recibieron el nombre formal de Brígatica el 1 de enero de 2002 (no hasta que fue declarado oficialmente como parte de un sitio oficial del Patrimonio Mundial de la UNESCO en 1990), fecha de la ceremonia de apertura formal de este pueblo. del presidente Joaquín Almunia, junto a otros muchos personajes destacados de la región al mismo tiempo. Brígatica tiene unos 5.000 habitantes y, según varias fuentes locales, hay entre 5.600 y 7.000. Incluso se está produciendo una especie de "guerra cultural"; Las tribus 'Briggatianas' parecen favorecer la creación de grandes poblaciones de asentamientos (pero de baja inmigración), a pesar de que Brígatica ha estado tradicionalmente dominada y gobernada por una cultura unifamiliar durante los últimos 150 años, una de las tribus más populares en una región conocida. colectivamente como Bragadin. A pesar del nombre, Bragadin no está oficialmente reconocido como una tribu nacional en el Congo. La otra cara de la moneda es que hay muchos grupos 'Briggatu' (ver más abajo): Existe una diferencia cultural obvia entre Brígatica y otras tribus Bragadin, y aunque todas las tribus Bragadin, independientemente de su origen, usan tradiciones tribales africanas tradicionales, Los bragazi modernos e industrializados también tienen creencias y formas de pensar muy diferentes (en términos de vestimenta), tanto tradicionalmente como en lo que respecta a'*

La función conditional_model presenta alguna serie de parámetros adicionales, que permiten controlar el tamaño de cada muestra o aumentar el número de muestras de salida. Sus opciones se pueden ver con:

    !conditinal_model??


## Conclusión

Como se puede observar tras probar algunas ejecuciones del programa, GPT-2 tiene una gran potencia para la generación automática de texto que, combinada con el soporte de re-entrenamiento, permite orientar el texto a un determinado tópico, manteniendo una cohesión gramatical aceptable.

Los resultados de este proyecto se podrían mejorar usando mucho más texto y entrenando durante mucho tiempo, pero eso no esuna opción factible dentro de Google Colab (y obtener un hardware potente para este reentrenamiento tampoco es barato precisamente). Aún así, la posibilidad existe y está más a nuestro alcance de lo que se podría esperar.
