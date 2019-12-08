# **Título**

Las trampas y los desafíos de aprendizaje automático fuera del Templo Kaggle

---

# **Descripción Corta**

El aprendizaje automático parece más un arte que una ciencia, y suele ser poco claro cómo se avanza desde un concurso de Kaggle hasta crear un proyecto desde cero. Tienes una idea genial, pero ¿cómo la transformas en una aplicación que puedas mostrar a tus amigos y disfrutar los chispazos de su envidia? Más importante aún ¿cuáles errores necesitas evitar para mantener la esperanza de sentir tales chispazos?

Vén para escuchar de mis pasos en falso, suposiciones expuestas y explotadas, y las sorpresas fastidiosas que acompañan implementar el aprendizaje automático en la realidad. Aprenderás sobre cómo lidiar con entrada de datos al azar, depender de la generosidad de desconocidos, manejar servidores de distintos tamaños y formas, y finalmente qué hacer cuando todo cambie, porque sin duda va a pasar.

Si te interesa el aprendizaje automático, y conoces las herramientas y técnicas básicas, pero tienes dudas de cómo lanzar una idea a producción, esta charla puede, por lo menos, mostrarte lo que no deberías hacer y cómo evitarlo.

---

_Resumen de qué trata la charla y por qué es interesante para los asistentes._

# **Descripción**

Esta charla es para la gente que conoce las herramientas populares del aprendizaje automático en Python, pero tiene poca experiencia con proyectos o aplicaciones del aprendizaje automático del mundo real. Esto incluye estudiantes, aficionados, y profesionales nuevos en la industria. Ellos y ellas conocen, y probablemente han usado, librerías como Numpy, Pandas, Scikit-Learn, y Tensorflow, comúnmente en el contexto de un proyecto de clase o concurso de Kaggle, en donde el problema está claramente definido, los datos provistos ya están limpios y formateados, y el ambiente de desarrollo es su propia computadora o una responsabilidad ajena.

De esta charla, la audiencia van a aprender de los desafíos y trampas que acompañan la creación de su propia applicación de aprendizaje automático desde cero. Así van a estar más preparados para evitar esos problemas y tener mejor probabilidad de producir un proyecto completo y exitoso.

**Esquema**

1. Introducción (2 mtos)
    * Explicar fútbol australiano, 'tipping' de fútbol, y la inspiración del proyecto.
    * Introducir la estructura de la charla: dos temporadas de trabajo con ejemplos de lo que hice mal en la primera, y cómo lo hice mejor en la segunda.
2. Entender el problema (2 mtos)
    * Primera temporada: creé un modelo para predecir los ganadores y perdedores.
        * No me di cuenta que hay que predecir los márgenes de victoria también.
    * Segunda temporada: creé un modelo para predecir los márgenes y ganadores.
3. Coleccionar los datos (5 mtos)
    * Primera temporada: escribí mi propio web scraper.
    * Segunda temporada: importé los datos de una librería que realizaba web scraping por mi.
        * Es mejor compartir el trabajo que trabajar solo.
        * Usar la herramienta correcta por el trabajo: importar datos de una librería de R puede ser menos trabajo que coleccionar los datos ti mismo.
    * Primera temporada: entré en pánico y cambié mi fuente de datos, porque el sitio no se actualizaba al principio de cada semana.
    * Segunda temporada: actualicé los datos de acuerdo al horario programado de los sitios web.
        * Además de entender los datos si mismos, es importante entender las fuentes de esos datos también.
4. Limpiar los datos (5 mtos)
    * Primera temporada: rellené y/o descarté datos en blanco y esperé que funcionara.
    * Segunda temporada: agregué muchas aserciones a propósito de la forma y el contenido de los datos para lanzar excepciones más temprano.
    * Afirmaciones explícitas de tus suposiciones pueden detener datos malformados que no causen errores de otra forma.
    * Dado fuentes dinámicas de datos, bugs no anticipados aparecen frequentemente y unit test estáticos no los encuentran.
5. Seleccionar el modelo (2 mtos)
    * Primera temporada: empecé por usar redes neuronales recurrentes de capas multiples, porque aprendizaje profundo es lo mejor ¿Cierto?
    * Segunda temporada: probé una variedad amplia de tipos de modelos, y las redes neuronales artificiales no tuvieron el mejor rendimiento.
    * Pon a prueba tus suposiciones, porque a veces los resultados te van a sorprender.
    * Interpretabilidad y mantenibilidad importan, porque esto no es algo que entregas solo por tener el mejor puntaje sin verlo nunca más.
6. La dicha de la producción (5 mtos)
    * Primera temporada: lancé la aplicación en Heroku justo antes del principio de la temporada, y no funcionó.
        * Algunas librerías de C no estaban disponibles en ese ambiente.
        * Lancé la aplicación al dentro de un contenedor de Docker para asegurarme de que todas dependencias estaban en el ambiente de producción.
    * Segunda temporada: lancé el contenidor en Heroku justo antes del principio de la temporada, y no funcionó.
        * El conjunto nuevo de datos de jugadores era demasiado grande y superó el límite de memoria del servidor gratuito.
        * Pagué un servidor más grande hasta que pudiera cambiar la architectura de la aplicación.
    * Conoce las diferencias entre el ambiente local y el ambiente de producción, los límites del segundo en particular.
7. Conclusión (2 mtos)
    * Primera temporada: a pesar de las equivocaciones, gané el concurso de 'tipping' de fútbol en my oficina por un solo punto.
    * Segunda temporada: a pesar de las mejoras, perdí el concurso de 'tipping' de fútbol en my oficina por cinco puntos.
    * Seguir buenas prácticas mejora la probabilidad, pero no garantiza la victoria.

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
    * [Post](https://dev.to/englishcraig/footy-tipping-with-machine-learning-2019-season-review-55mc) que resume el rendimiento de 2019 y presenta los planes para el futuro.

---

_Información general sobre el ponente para los revisores. Experiencia previa, dominio del tema, etc._
