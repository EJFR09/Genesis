---
status: open
priority: normal
scheduled:
dateCreated: 2026-10-20T19:59:31.090-03:00
dateModified: 2026-10-20T19:59:31.090-03:00
tags:
  - task
---
# TP3 — Plan de trabajo

**Materia:** Diseño de Compiladores  
**Proyecto:** Análisis de conversaciones sobre reservas y turnos  
**Entrega:** 19/11/2026  
**Integrantes:** 2 + IA

---

# FASE 1 — Diseño formal y documentación de la gramática

**Objetivo:** Definir el lenguaje admitido, construir sus expresiones regulares y autómatas, desarrollar una gramática predictiva y establecer las reglas semánticas antes de implementar el programa.

## 1.1. Análisis y especificación del lenguaje

- [ ] G01. Analizar los requisitos del enunciado y los ocho casos guía.
- [ ] G02. Identificar todas las construcciones lingüísticas admitidas: reservas, confirmaciones, cambios, cancelaciones y valoraciones.
- [ ] G03. Definir el formato de entrada: sesiones, turnos, roles, numeración y delimitadores.
- [ ] G04. Establecer los límites del lenguaje y las construcciones no admitidas.
- [ ] G05. Definir el vocabulario canónico, sinónimos y equivalencias.
- [ ] G06. Documentar ejemplos de cadenas válidas e inválidas para cada construcción.
## 1.2. Diseño del analizador léxico

- [ ] G07. Identificar los lexemas, tokens y patrones necesarios.
- [ ] G08. Definir las categorías de tokens y sus prioridades.
- [ ] G09. Especificar las expresiones regulares de cada categoría.
- [ ] G10. Definir las reglas de reconocimiento de referencias RS, días y horarios.
- [ ] G11. Seleccionar al menos una expresión regular para desarrollar formalmente su autómata.
- [ ] G12. Construir el AFN mediante el procedimiento de Thompson.
- [ ] G13. Convertir el AFN a AFD mediante el algoritmo de subconjuntos y cerradura-ε.
- [ ] G14. Construir la tabla de transiciones y el diagrama del AFD.
- [ ] G15. Analizar posibles ambigüedades léxicas y definir sus prioridades de resolución.
- [ ] G16. Documentar las reglas de normalización y protección de tokens especiales.

## 1.3. Diseño de la gramática BNF

- [ ] G17. Definir los símbolos terminales, no terminales y el símbolo inicial.
- [ ] G18. Construir la BNF inicial del formato de sesiones y turnos.
- [ ] G19. Construir las producciones para solicitudes, confirmaciones, cambios, cancelaciones y valoraciones.
- [ ] G20. Incorporar las construcciones de negación (`no`), intensificación (`muy`) y contraste (`pero`).
- [ ] G21. Identificar posibles ambigüedades sintácticas.
- [ ] G22. Eliminar la recursión por la izquierda cuando corresponda.
- [ ] G23. Eliminar factores comunes directos e indirectos.
- [ ] G24. Obtener la gramática predictiva final.
- [ ] G25. Calcular los conjuntos PRIMERO (PRIM).
- [ ] G26. Calcular los conjuntos SIGUIENTE (SGTE).
- [ ] G27. Construir la tabla de análisis sintáctico M.
- [ ] G28. Comprobar y justificar que la gramática resultante sea LL(1), resolviendo los conflictos que aparezcan.
- [ ] G29. Realizar derivaciones y construir árboles sintácticos de prueba.
- [ ] G30. Verificar que las transformaciones no incorporen cadenas inválidas ni excluyan construcciones obligatorias.

## 1.4. Diseño semántico y estructuras de datos

- [ ] G31. Definir los atributos y acciones semánticas asociados a las producciones.
- [ ] G32. Identificar atributos sintetizados y heredados cuando corresponda.
- [ ] G33. Definir las reglas de interpretación de valoraciones por aspecto.
- [ ] G34. Especificar el alcance semántico de la negación, intensificación y contraste.
- [ ] G35. Definir los estados conversacionales y sus reglas de transición.
- [ ] G36. Especificar las condiciones para actualizar el turno vigente y la solicitud pendiente.
- [ ] G37. Diseñar la estructura y el ámbito de la tabla de símbolos.
- [ ] G38. Diseñar la representación sintáctica intermedia de las cláusulas reconocidas.
- [ ] G39. Definir los contratos de entrada y salida entre lexer, parser y analizador semántico.
- [ ] G40. Definir la estructura esperada del JSON y de las evidencias.
- [ ] G41. Diseñar las políticas de corrección ortográfica, abstención y recuperación de errores.

## 1.5. Herramientas y documentación formal

- [ ] G42. Investigar y comparar LaTeX, JFLAP, Graphviz, Mermaid y otras herramientas para crear, verificar y documentar digitalmente gramáticas y autómatas.
- [ ] G43. Seleccionar los formatos y herramientas definitivos.
- [ ] G44. Organizar la documentación de expresiones regulares, AFN, AFD y sus procedimientos de construcción.
- [ ] G45. Documentar la BNF inicial, las transformaciones y la gramática predictiva final.
- [ ] G46. Documentar PRIM, SGTE, tabla M y reglas semánticas.
- [ ] G47. Revisar conjuntamente toda la especificación formal antes de iniciar la implementación.

---

# FASE 2 — Implementación, integración y entrega

**Objetivo:** Implementar el procesador determinista de texto a partir de la especificación formal, validar su funcionamiento y preparar todos los materiales de entrega y defensa.

## 2.1. Preparación del proyecto y colaboración

- [ ] I01. Elegir el lenguaje de programación y el entorno de desarrollo.
- [ ] I02. Crear el repositorio de GitHub e invitar a la compañera.
- [ ] I03. Definir la estrategia de ramas, Pull Requests y revisiones.
- [ ] I04. Definir convenciones de código, commits y documentación.
- [ ] I05. Preparar la estructura de carpetas del proyecto.
- [ ] I06. Configurar las dependencias y el entorno de ejecución.
- [ ] I07. Organizar las tareas mediante GitHub Issues y Projects.
- [ ] I08. Distribuir responsabilidades y establecer revisiones cruzadas.
- [ ] I09. Configurar OpenCode con las instrucciones y restricciones del proyecto.
- [ ] I10. Preparar la estructura inicial de pruebas automatizadas.

## 2.2. Implementación del analizador léxico

- [ ] I11. Implementar la lectura de archivos, sesiones y turnos.
- [ ] I12. Implementar la tokenización según las expresiones regulares definidas.
- [ ] I13. Implementar el reconocimiento de referencias RS, días y horarios.
- [ ] I14. Implementar la normalización y conservar los lexemas originales.
- [ ] I15. Registrar posiciones de sesión, turno y desplazamiento.
- [ ] I16. Implementar la política de prioridad entre tokens.
- [ ] I17. Implementar el algoritmo de distancia de Levenshtein.
- [ ] I18. Corregir automáticamente solo candidatos únicos a distancia 1.
- [ ] I19. Proteger negaciones, referencias, horarios, días y cifras contra correcciones indebidas.
- [ ] I20. Detectar empates y registrar los casos en los que se decide no corregir.
- [ ] I21. Implementar pruebas unitarias del analizador léxico y la corrección.

## 2.3. Implementación del analizador sintáctico

- [ ] I22. Implementar el parser predictivo conforme a la BNF final.
- [ ] I23. Implementar los procedimientos correspondientes a los no terminales.
- [ ] I24. Implementar el consumo y la validación de tokens mediante `match`.
- [ ] I25. Construir la representación sintáctica de las cláusulas reconocidas.
- [ ] I26. Preservar la negación, intensificación y contraste en las estructuras generadas.
- [ ] I27. Reconocer las distintas construcciones lingüísticas admitidas.
- [ ] I28. Identificar fragmentos `NO_INTERPRETADOS`.
- [ ] I29. Implementar pruebas unitarias del parser con entradas válidas, inválidas y parciales.

## 2.4. Implementación de la tabla de símbolos

- [ ] I30. Implementar las estructuras de la tabla de símbolos.
- [ ] I31. Implementar las operaciones de inserción, búsqueda y actualización.
- [ ] I32. Establecer el ámbito de cada sesión.
- [ ] I33. Implementar el control de duplicados.
- [ ] I34. Enlazar menciones repetidas de una reserva a una sola entrada.
- [ ] I35. Registrar participantes, apariciones, turno vigente y solicitudes pendientes.
- [ ] I36. Mantener separado el diccionario de vocabulario.
- [ ] I37. Implementar pruebas unitarias de la tabla de símbolos.

## 2.5. Implementación del análisis semántico

- [ ] I38. Implementar las acciones semánticas correspondientes a las producciones.
- [ ] I39. Determinar el actor y la intención de cada cláusula.
- [ ] I40. Identificar aspectos y valoraciones: POSITIVA, NEGATIVA, NEUTRA, MIXTA y SIN_EVIDENCIA según corresponda.
- [ ] I41. Aplicar las reglas de negación, intensificación y contraste.
- [ ] I42. Implementar las transiciones de estado: CONFIRMADA, REPROGRAMADA, CANCELADA, PENDIENTE y SIN_CONFIRMAR.
- [ ] I43. Conservar el turno vigente mientras exista una solicitud de cambio pendiente.
- [ ] I44. Actualizar el turno vigente únicamente cuando exista evidencia explícita de ejecución.
- [ ] I45. Registrar evidencias con turno, fragmento original y regla aplicada.
- [ ] I46. Generar las valoraciones por turno y el resumen por aspecto.
- [ ] I47. Implementar pruebas unitarias de las reglas semánticas y los estados.

## 2.6. Integración y recuperación de errores

- [ ] I48. Integrar el analizador léxico, parser, tabla de símbolos y análisis semántico.
- [ ] I49. Implementar el rechazo de turnos con roles inválidos u orden inconsistente.
- [ ] I50. Recuperar el procesamiento desde la siguiente línea válida.
- [ ] I51. Garantizar la continuidad entre sesiones después de encontrar errores.
- [ ] I52. Implementar la generación del JSON final.
- [ ] I53. Implementar la presentación legible de resultados en consola.
- [ ] I54. Implementar la traza completa: texto → tokens → estructura → tabla de símbolos → acción semántica → resultado.
- [ ] I55. Verificar que los resultados sean reproducibles.

## 2.7. Pruebas y validación

- [ ] I56. Preparar los ocho casos guía como sesiones completas.
- [ ] I57. Diseñar cuatro casos adicionales propios.
- [ ] I58. Documentar las salidas esperadas de cada caso.
- [ ] I59. Ejecutar los casos y registrar las salidas obtenidas.
- [ ] I60. Comparar automáticamente los resultados esperados y obtenidos.
- [ ] I61. Verificar negaciones, contrastes y evidencia contradictoria.
- [ ] I62. Verificar corrección ortográfica y abstención ante empates.
- [ ] I63. Verificar recuperación de errores y procesamiento de múltiples sesiones.
- [ ] I64. Incorporar al menos una prueba de regresión.
- [ ] I65. Ejecutar pruebas de integración después de los cambios.
- [ ] I66. Configurar GitHub Actions para ejecutar automáticamente las pruebas.
- [ ] I67. Validar el comportamiento ante casos no vistos previamente dentro del lenguaje admitido.

## 2.8. Documentación, entrega y defensa

- [ ] I68. Redactar el informe técnico con los procedimientos y decisiones de diseño.
- [ ] I69. Documentar la implementación y su correspondencia con la gramática.
- [ ] I70. Preparar el README con instrucciones de instalación y ejecución.
- [ ] I71. Documentar dependencias, bibliotecas y herramientas utilizadas.
- [ ] I72. Documentar el uso de IA y las contribuciones de cada integrante.
- [ ] I73. Incorporar los casos de prueba, salidas esperadas y resultados obtenidos.
- [ ] I74. Preparar un JSON de ejemplo y una traza de procesamiento completa.
- [ ] I75. Verificar que el proyecto funcione desde una instalación limpia.
- [ ] I76. Preparar la explicación de los algoritmos y estructuras empleados.
- [ ] I77. Ensayar la defensa individual y la modificación de una regla.
- [ ] I78. Realizar la revisión final de requisitos y criterios de evaluación.
- [ ] I79. Preparar la versión definitiva de entrega.

---

## Criterios para cerrar cada fase

### Fase 1 — Gramática

- El lenguaje y su vocabulario están delimitados y documentados.
- Las expresiones regulares y los autómatas están justificados.
- La BNF reconoce las construcciones obligatorias.
- La gramática predictiva está validada con PRIM, SGTE y tabla M.
- Las reglas semánticas, estructuras y contratos están definidos.
- Ambos integrantes revisaron y aprobaron la especificación.

### Fase 2 — Implementación

- [ ] Todos los componentes obligatorios funcionan e interactúan correctamente.
- [ ] Se superan los ocho casos guía y los cuatro casos propios.
- [ ] Existe recuperación ante errores y evidencia trazable.
- [ ] Se genera un JSON y una salida legible en consola.
- [ ] El proyecto se ejecuta de manera reproducible.
- [ ] El código, la documentación y las pruebas están completos.
- [ ] Ambos integrantes pueden explicar y modificar el programa durante la defensa.