---
status: open
priority: normal
scheduled:
dateCreated: 2026-10-20T19:59:31.090-03:00
dateModified: 2026-10-20T19:59:31.090-03:00
tags:
  - task
---

Fase de Gramatica
![[Pasted image 20261020200323.png]]


FASE 0: Preparación y organización
* [ ] 00.1 Elegir lenguaje de programación y entorno
- [ ] 00.2 Crear repositorio e invitar a la compañera
- [ ] 00.3 Definir ramas, revisiones y convenciones
- [ ] 00.4 Preparar estructura de carpetas y tareas
- [ ] 00.5 Definir contratos de entrada y salida entre módulos

FASE 1: Especificación del lenguaje
- [ ] 01.1 Identificar construcciones lingüísticas admitidas
- [ ] 01.2 Definir vocabulario canónico y sinónimos
- [ ] 01.3 Diseñar categorías de tokens y prioridades
- [ ] 01.4 Especificar expresiones regulares
- [ ] 01.5 Construir y documentar al menos un AFD
- [ ] 01.6 Diseñar y revisar la gramática BNF
- [ ] 01.7 Definir negación, intensificación y contraste
- [ ] 01.8 Establecer estructura sintáctica intermedia
- [ ] 01.9 Justificar PRIM, SGTE y LL(1), si corresponde

FASE 2A: Analizador léxico y corrección
- [ ] 02.1 Leer sesiones, turnos y posiciones
- [ ] 02.2 Implementar tokenización y normalización
- [ ] 02.3 Reconocer referencias RS, días y horarios
- [ ] 02.4 Implementar Levenshtein
- [ ] 02.5 Proteger tokens especiales y detectar empates
- [ ] 02.6 Registrar correcciones y probar el lexer

FASE 2B: Parser
- [ ] [ ] 03.1 Implementar reconocimiento de producciones BNF
- [ ] [ ] 03.2 Construir representación de cláusulas
- [ ] [ ] 03.3 Preservar negación, muy y pero
- [ ] [ ] 03.4 Clasificar construcciones sintácticas
- [ ] [ ] 03.5 Marcar fragmentos NO_INTERPRETADOS
- [ ] [ ] 03.6 Probar casos válidos y parciales

FASE 3: Tabla de símbolos y semántica
- [ ] [ ] 04.1 Diseñar entradas y ámbito de la tabla
- [ ] [ ] 04.2 Implementar inserción, búsqueda y actualización
- [ ] [ ] 04.3 Enlazar referencias repetidas y apariciones
- [ ] [ ] 04.4 Definir reglas y atributos semánticos
- [ ] [ ] 04.5 Determinar actor, aspecto y polaridad
- [ ] [ ] 04.6 Construir resumen de evidencias por aspecto
- [ ] [ ] 04.7 Implementar transiciones de estado
- [ ] [ ] 04.8 Separar turno vigente y solicitud pendiente

FASE 4: Integración y recuperación
- [ ] [ ] 05.1 Conectar lexer, parser y semántica
- [ ] [ ] 05.2 Rechazar roles y órdenes inválidos
- [ ] [ ] 05.3 Continuar tras errores sin perder sesiones
- [ ] [ ] 05.4 Diseñar y generar JSON de salida
- [ ] [ ] 05.5 Crear vista legible en consola
- [ ] [ ] 05.6 Generar traza de procesamiento completa

FASE 5: Pruebas, entrega y defensa
- [ ] [ ] 06.1 Convertir los ocho casos guía en sesiones
- [ ] [ ] 06.2 Diseñar cuatro casos propios
- [ ] [ ] 06.3 Registrar resultados esperados y obtenidos
- [ ] [ ] 06.4 Agregar prueba de regresión
- [ ] [ ] 06.5 Probar múltiples sesiones y casos límite
- [ ] [ ] 06.6 Redactar informe técnico
- [ ] [ ] 06.7 Preparar README, comando y dependencias
- [ ] [ ] 06.8 Documentar contribuciones y uso de IA
- [ ] [ ] 06.9 Ensayar traza y modificación de regla
- [ ] [ ] 06.10 Realizar revisión final de la entrega
