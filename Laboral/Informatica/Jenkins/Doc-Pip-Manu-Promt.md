Genera un manual de usuario para un pipeline de Jenkins en formato Markdown.
El documento debe seguir esta estructura y convenciones:

* Título principal con el nombre del proyecto entre corchetes y el tipo de pipeline
* Sección de metadatos: autores, links a pipelines y links a repositorios
* Introducción breve: qué hace el pipeline, qué tipo de job de Jenkins usa, y qué función/archivo lo implementa
* Sección "Cómo funciona": cómo se dispara manualmente, qué parámetros hay que completar (obligatorios y opcionales/condicionales), en qué orden hacerlo, y qué stages hay que monitorear
* Resumen general en lista: tipo, uso, tecnología y flujo de trabajo resumido
* Flujo de trabajo detallado por stage: descripción breve + bloque de código relevante, indicando qué stages son condicionales y bajo qué condición corren
* Configuración requerida en cada repositorio: qué archivos, tasks de Gradle/Maven, plugins o convenciones de estructura necesita el repo que consume el pipeline, con bloques de código de ejemplo
* Consideraciones finales: solo lo que no se haya dicho antes (comportamientos especiales, diferencias con otros pipelines similares, credenciales que deben estar preconfiguradas en Jenkins)

Convenciones:

* Los nombres de comandos, parámetros, funciones y archivos siempre en backticks
* Los bloques de código con el lenguaje indicado (groovy, kotlin, properties, etc.)
* Nada de redundancias: si algo ya se explicó, no repetirlo
* Tono técnico pero directo, sin relleno
* Si el pipeline tiene parámetros condicionales (por ejemplo un campo que solo se habilita si otro parámetro tiene cierto valor), documentar explícitamente esa dependencia
* Si el pipeline construye/publica múltiples artefactos o imágenes de forma selectiva según checkboxes, explicar cómo se arma esa selección internamente