# **Título**

Las trampas y los desafíos de aprendizaje automático afuera del claustro de Kaggle

---

# **Descripción Corta**

El aprendizaje automático parece más un arte que una ciencia, y suele ser poco claro cómo se avanza desde un concurso de Kaggle hasta crear un proyecto desde cero. Tienes una idea genial, pero ¿cómo la transformas en una aplicación que puedas mostrar a tus amigos para disfrutar los rayos calientes de su envídia? Más importante aún ¿cuales errores necesitas evitar para mantener la esperanza de sentir tales rayos?

Vén para escuchar mis equivocaciones, suposiciones expuestas y explotadas, y las sorpresas desagradables que acompañan implementar el aprendizaje automático en los salvajes. Aprenderán soportar la entrada al azar de datos, depender de la generosidad de desconocidos, manejar las cajas de formas y tamaños diferentes, y finalmente qué hacer cuando todo cambie, porque sin duda va a pasar.

Si te interesa el aprendizaje automático, y conoces las herramientas y los técnicos básicos, pero tienes dudas a propósito de lanzar una idea para un proyecto a la producción, esta charla puede, por lo menos, mostrarte lo que no deberías hacer y cómo no hacerlo.

---

_Resumen de qué trata la charla y por qué es interesante para los asistentes._

# **Descripción**

Esta charla is para la gente que conoce las herramientas populares del aprendizaje automático en Python, pero tiene poca experiencia con proyectos o applicaciones del aprendizaje automático del mundo real. Se incluyen los estudiantes, los aficionados, y los profesionales recién llegados a esta indústria. Miembros de la audiencia objectiva conocen, y es probable que hayan usado, librerías de código como Numpy, Pandas, Scikit-Learn, y Tensorflow, pero en el contexto de un proyecto de clase or concurso de Kaggle, en que se define claramente el problema, los datos provistos ya están limpios y formateados, y el ambiente es su propia computadora o una responsibilidad ajena.

De esta charla, la audiencia van a aprender de los desafíos y trampas que acompañan la creación de su propia applicación de aprendizaje automático desde cero. Así van a estar más preparados para evitar esos problemas y tener mejor probabilidad de producir un proyecto completo y exitoso.

**Esquema**

1. Introducción (2 mtos)
    * Explicar fútbol australiano, 'tipping' de fútbol, y la inspiración del proyecto.
    * Introducir la estructura de la charla: dos temporadas de trabajo con ejemplos de lo que hice mal en la primera, y cómo lo hice mejor en la segunda.
2. Entender el problema (2 mtos)
    * Primera temporada: creé un modelo para predecir los ganadores y perdedores.
        * No me di cuenta que hay que predecir los márgenes de vitoria también.
    * Segunda temporada: creé un modelo para predecir los márgenes y ganadores.
3. Coleccionar los datos (5 mtos)
    * Primera temporada: escribí mi propio web scraper.
    * Segunda temporada: importé los datos de una librería que raspaba los sitios para mi.
        * Es mejor compartir el trabajo que trabajar solo.
        * Usar la herramienta correcta por el trabajo: importar datos de una librería de R puede ser menos trabajo que coleccionar los datos ti mismo.
    * Primera temporada: tuve un pánico y cambié mi fuente de datos, porque el sitio no se actualizaba al principio de cada semana.
    * Segunda temporada: actualizaba los datos de acuerdo con el horario de actualización de los sitios web.
        * Además de entender los datos si mismos, es importante entender las fuentes de esos datos también.
4. Limpiar los datos (5 mtos)
    * Primera temporada: rellenaba y/o sacaba datos blancos y esperaba que funcionara.
    * Segunda temporada: agregué muchas aserciones a propósito de la forma y el contenido de los datos para lanzar excepciones más temprano.
    * Afirmaciones explícitas de las suposiciones pueden detener datos malformados que no causen errors de otra forma.
    * Dado fuentes dinámicas de datos, bichos no anticipados aparecen frequentemente y pruebas estáticas de unidad no los encuentran.
5. Seleccionar el modelo (2 mtos)
    * Primera temporada: empecé por usar redes neuronales recurrentes de capas multiples, porque aprendizaje profundo es lo mejor ¿no?
    * Segunda temporada: probé una variedad amplia de tipos de modelos, y las redes neuronales artificiales no tuvieron el mejor rendimiento.
    * Prueba tus suposiciones, porque a veces los resultados te van a sorprender.
    * Interpretabilidad y mantenibilidad importan, porque este no es un modelo que entregues para la mejor métrica sin verlo nunca más.
6. La alegría de la producción (5 mtos)
    * Primera temporada: lancé la aplicación en Heroku justo antes del principio de la temporada, y no funcionó.
        * Algunas librerías de C no estaban disponibles en ese ambiente.
        * Lancé la aplicación al dentro de un contenedor de Docker para asegurarme de que todas dependencias estaban en el ambiente de producción.
    * Segunda temporada: lancé el contenidor en Heroku justo antes del principio de la temporada, y no funcionó.
        * El conjunto nuevo de datos de jugadores era demasiado grande y superó el límite de memoria del servidor gratis.
        * Pagué un servidor más grande hasta que pudiera cambiar la architectura de la aplicación.
    * Conoce las diferencias entre el ambiente local y el ambiente de producción, los límites del segundo en particular.
7. Conclusión (2 mtos)
    * Primera temporada: a pesar de las equivocaciones, gané el concurso de 'tipping' de fútbol en my oficina por un solo punto.
    * Segunda temporada: a pesar de los mejoramientos, perdí el concurso de 'tipping' de fútbol en my oficina por cinco puntos.
    * Seguir buenas prácticas mejora la probabilidad, pero no guarantiza la vitoria.

---

_Cual es el tema de la charla, que va a cubrir y por qué esta charla debería ser elegida para las PyCon Charlas._

# **Additional Notes**

* Código del modelo de la primera temporada: [FootyTipper](https://github.com/cfranklin11/footy-tipper)
* Código del modelo de la segunda temporada:
    * Main application: [Tipresias](https://github.com/tipresias/tipresias)
    * Data processing and machine learning models: [Augury](https://github.com/tipresias/augury)
    * Data source: [BirdSigns](https://github.com/tipresias/bird-signs)
* Una muestra de posts de blog relevantes:
    * [Post](https://medium.com/@craigjfranklin/toward-a-better-footy-tipping-model-mistakes-were-made-ee5a6738741f) que introduce la idea y el modelo.
    * [Post](https://medium.com/@craigjfranklin/footy-tipping-with-machine-learning-models-assemble-5f884a7e8538) sobre la selección del modelo.
    * [Post](https://dev.to/englishcraig/footy-tipping-with-machine-learning-2019-season-review-55mc) que resume el rendimiento de 2019 y presenta los planes para el futuro..

---

_Información general sobre el ponente para los revisores. Experiencia previa, dominio del tema, etc._
