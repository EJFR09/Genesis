Genera un manual de usuario para un pipeline de Jenkins en formato Markdown.
El documento debe seguir esta estructura y convenciones:

- Título principal con el nombre del proyecto entre corchetes y el tipo de pipeline
- Sección de metadatos: autores, links a pipelines y links a repositorios
- Introducción breve: qué hace el pipeline, qué tipo de job de Jenkins usa, y qué función/archivo lo implementa
- Sección "Cómo funciona": cómo se dispara, qué stages hay que monitorear, y qué parámetros opcionales acepta
- Resumen general en lista: tipo, uso, tecnología y flujo de trabajo resumido
- Flujo de trabajo detallado por stage: descripción breve + bloque de código relevante
- Configuración requerida en cada repositorio: qué archivos o ajustes necesita el repo que consume el pipeline, con bloques de código de ejemplo
- Consideraciones finales: solo lo que no se haya dicho antes

Convenciones:
- Los nombres de comandos, parámetros, funciones y archivos siempre en backticks
- Los bloques de código con el lenguaje indicado (groovy, kotlin, properties, etc.)
- Las advertencias importantes con > OBS: ...
- Nada de redundancias: si algo ya se explicó, no repetirlo
- Tono técnico pero directo, sin relleno