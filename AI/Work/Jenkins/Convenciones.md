# Convenciones para pipelines de Jenkins

> Guía de trabajo extraída de los pipelines de DNIT SID API y Backoffice. Distingue prácticas reutilizables de decisiones particulares de estos repositorios; no es una norma global para todos los proyectos.

## 1. Separar CI de release

- **Build/CI:** job multibranch disparado por cambios en Git. Valida código, pruebas y calidad; no publica imágenes, no versiona ni despliega.
- **Release:** job manual con parámetros. Valida primero, publica artefactos/imágenes y, si se solicita, despliega. Versiona el repositorio y crea el tag Git.
- Mantener la lógica en una Jenkins Shared Library, en `vars/<nombre>.groovy`. El `Jenkinsfile` del repositorio debe ser pequeño: cargar `@Library('dnit-sid') _` y llamar a la función correspondiente.
- En CI usar `disableConcurrentBuilds(abortPrevious: true)` cuando un push nuevo vuelve obsoleto al build anterior. En release evitar ejecuciones simultáneas del mismo job con `disableConcurrentBuilds()`.
- Declarar herramientas y recursos del agente (`jdk21`, `nodejs`, Docker, SonarScanner) de forma explícita.

## 2. Contrato del repositorio consumidor

Antes de crear el pipeline, identificar estructura, scripts y tareas reales del proyecto. No asumir que la raíz produce un artefacto ejecutable.

**Gradle multimódulo (SID API):**

- El wrapper `./gradlew` es la entrada única; el proyecto usa Java 21.
- `gradle.properties` contiene `projectVersion`; todos los módulos heredan la versión. La tarea raíz `./gradlew -q printVersion` la expone al pipeline.
- Cada aplicación ofrece `bootJar`; `sid-common` es una librería, no una imagen.
- Las aplicaciones con SonarQube tienen claves independientes. El Dockerfile acepta `APP_PORT` y `JAR_FILE`, con el módulo como contexto.
- `settings.gradle.kts` consulta `mavenLocal()` antes de repositorios remotos cuando debe resolver artefactos Michi construidos en el job.

**Frontend npm (SID Backoffice):**

- Cada carpeta (`sid-dnit-backoffice`, `sid-bank-backoffice`) tiene `package.json`, `package-lock.json` y scripts `lint`, `typecheck`, `test` y `build`.
- Antes de instalar/compilar, el pipeline escribe un `.env` por aplicación con `VITE_API_URL=/sid-taxinv`. Es una configuración de build, no un secreto.
- Usar `npm ci` para instalar desde el lockfile.
- El Dockerfile raíz recibe `APP_DIR` y reconstruye lo necesario dentro de la imagen: el release elimina `node_modules` y `dist` del workspace antes de `docker build`.

## 3. Parámetros y selección

- Validar en `Prepare`: rama destino, tipo de incremento, comentario del tag, referencias externas necesarias y al menos un componente seleccionado.
- Representar los componentes en un mapa con checkbox, ruta/módulo, imagen, clave Sonar, puerto o manifiesto. Filtrar ese mapa una sola vez; usar la selección resultante en build, Sonar, Docker y deploy.
- `BUMP_TYPE`: `patch`, `minor`, `major` u `other`. Los campos de versión manual solo aparecen y se usan con `other` y el componente correspondiente seleccionado.
- `DEPLOY` es booleano y solo actúa sobre componentes seleccionados. `DRY_RUN` refresca el formulario de Jenkins; no debe compilar, publicar, desplegar ni crear commits/tags.
- En CI, los refs de una dependencia externa pueden entrar por argumentos del `Jenkinsfile` con defaults; en el release de SID API se exponen como parámetros manuales con defaults.

## 4. Dependencias Git externas y reproducibilidad

El módulo `sid-oidc` necesita artefactos de una referencia Michi distinta de la usada por `sid-auth`, `sid-taxinv` y `sid-consent`.

- Mantener dos referencias: general (`oficial/v11/devel`) y OIDC (`oficial/v11/michi-oauth-oidc-dnit`), ambas reemplazables. Admitir rama, `refs/tags/<tag>` o SHA mediante `checkout scmGit`; registrar el SHA realmente obtenido.
- Publicar `michi-common`, `michi-common-client` y `michi-auth` desde la referencia general. Para OIDC, publicar `michi-common`, `michi-common-client` y `michi-oauth2-oidc` desde la otra.
- Aislar repositorios Maven bajo el workspace: `.m2/general/repository` y `.m2/oidc/repository`. Ejecutar build/test/Sonar de cada módulo con su `-Dmaven.repo.local` correspondiente. No depender del `~/.m2` global del agente.
- Usar `--no-daemon` en estos jobs para no reutilizar un daemon Gradle entre builds. Aun así Gradle puede iniciar un daemon de uso único; ese mensaje no indica por sí solo un error.
- Fijar un SHA cuando sea esencial repetir exactamente un release; una rama puede avanzar.

## 5. Orden de CI y calidad

- `Prepare`: cargar helpers, registrar contexto y leer versión. Si el asunto del commit contiene `chore: jenkins bump version`, omitir stages de trabajo para no repetir CI por el commit automático de release; el `post` sigue corriendo.
- `Build` y `Test` separados. En SID API se compila con `-x test` y luego se ejecutan pruebas con timeout de 30 minutos. En Backoffice se ejecutan `lint`, `typecheck`, tests y build de ambas aplicaciones; el test acepta `--passWithNoTests`.
- Ejecutar Sonar por aplicación, no como un único análisis agregado cuando cada módulo es un proyecto Sonar distinto. Después de cada scanner invocar `waitForQualityGate abortPipeline: true` con timeout (cinco minutos en los builds actuales).
- Si Jenkins advierte sobre varios `report-task.txt`, verificar que cada `waitForQualityGate` esté asociado al análisis correcto; no dar por validados todos los proyectos solo por una espera exitosa.
- El bloque `post { always { ... } }` debe correr incluso con fallas o skips. Confirmar lo que hace realmente el helper: en la implementación observada imprime datos de resultado y ejecuta `cleanWs()`; los envíos habituales de Slack/correo están comentados.

## 6. Orden de release y versionado

Secuencia usada: `Prepare` → calcular versión → preparar dependencias → validar/compilar → Sonar y Quality Gate → Docker build/push → deploy opcional → commit de versión → tag Git → `post`.

- **SID API:** una versión compartida desde `printVersion`. Pasar la nueva versión a Gradle con `-PprojectVersion` al construir, antes de modificar `gradle.properties`. El tag Git es `<versión>-<BUILD_NUMBER>`.
- **SID Backoffice:** cada aplicación tiene versión independiente en `package.json`. Un bump se calcula desde la versión de cada aplicación seleccionada; no se exige que coincidan. `npm version --no-git-tag-version --allow-same-version` permite repetir la misma versión sin fallar.
- Si no hay cambios de versión, no crear un commit vacío. Si los hay, hacer `git fetch`, `git rebase` y push a la rama indicada; no usar force push. Para el commit automático de Backoffice se desactiva Husky con `HUSKY=0`.
- Un release de Backoffice crea **un solo tag para el estado del repositorio**, siempre con ambas versiones, aunque solo se haya publicado una imagen: `sid-backoffice--dnit-<versión>--bank-<versión>--<BUILD_NUMBER>`. La versión del componente no seleccionado se lee de su `package.json`. No usar corchetes literales en nombres de tags: Git los rechaza.
- El tag se crea después del commit; puede crearse incluso si no hubo diferencias para commitear. Documentar esa semántica al consumidor.

## 7. Imágenes y despliegue

- Construir y publicar solo imágenes seleccionadas. En estos pipelines el registry configurado es Dev; no prometer selección Prod si no existe en el código.
- Publicar cada imagen con su versión y `latest`. `latest` es mutable: para trazabilidad y rollback usar el tag versionado.
- `DEPLOY` no se ejecuta si no se seleccionó ningún componente. En OKD, autenticar cada sesión SSH con credenciales Jenkins y `okd-login`; nunca guardar contraseñas en scripts o notas.
- Actualizar primero todos los manifiestos seleccionados y ejecutar un `oc apply` con todos los `-f`. Esto evita que un rollout lento del primer componente impida aplicar el resto; **no convierte el apply en una transacción atómica**.
- Comprobar rollouts en paralelo, hasta 180 segundos por deployment. Si falla el apply, reintentar una vez y detener el release si vuelve a fallar. Si falla la comprobación de rollout, marcar `UNSTABLE` y continuar con commit/tag. `UNSTABLE` no equivale a despliegue saludable.
- Un `connection refused` en la probe indica que la app aún no escucha en el puerto esperado; revisar logs, variables/ConfigMaps, puerto, contexto y probes del pod. No atribuirlo automáticamente al pipeline.
- El rollback del deployment y la corrección del YAML almacenado son asuntos separados: revertir una revisión en OKD no necesariamente restaura el manifiesto editado fuera del cluster.

## 8. Seguridad y trazabilidad

- Usar IDs de credenciales preconfiguradas para Git, Docker y OKD; no escribir tokens ni contraseñas en el repositorio, parámetros visibles o logs. Ocultar trazas de shell alrededor de autenticación.
- Validar temprano versiones y nombres de tags. Un fallo tardío en Git puede dejar imágenes ya publicadas; la publicación, el apply y el tag **no son atómicos**.
- Registrar versión, componentes seleccionados, refs/SHA de dependencias, imágenes y tag generado. Mantener el tag Git y las imágenes rastreables al mismo commit.
- Recordar que en Backoffice la actualización de `package.json` ocurre después del build de la imagen: si la app muestra su versión interna, verificar que coincida con la etiqueta Docker.
- Revisar los casos de una sola aplicación seleccionada, ambas, versión repetida, `other`, `DRY_RUN`, Quality Gate rechazado, apply fallido y rollout `UNSTABLE` antes de dar un pipeline por terminado.

## 9. Documentación mínima por pipeline

Crear un manual Markdown con: autor, links de job/repos, activación, parámetros obligatorios/condicionales, selección de componentes, stages y sus `when`, comportamiento ante fallas, contrato del repositorio, herramientas/credenciales requeridas y alcance (qué hace y qué no). Mantener el manual sincronizado con el Groovy; no documentar notificaciones o métricas que el helper tenga comentadas.

**Fuentes de esta guía:** `pipelines/dnit-sid/vars/sidapibuild.groovy`, `sidapirelease.groovy`, `sidbackofficebuild.groovy` y `sidbackofficerelease.groovy` del repositorio DevOps; decisiones y fallos discutidos durante su puesta en marcha.