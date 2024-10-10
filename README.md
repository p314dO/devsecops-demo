# How to build a DevSecOps Pipeline in GitHub

### build.yml

Este archivo define un "job" llamado `build`, que se ejecuta en un entorno de Ubuntu para compilar un proyecto Java usando Maven, realizar pruebas, y subir los artefactos resultantes.

### Desglose del Job

1. **Nombre del Job**: 
   ```yaml
   build:
   ```
   Se define un job llamado `build`.

2. **Permisos**:
   ```yaml
   permissions:
     contents: read
     issues: read
     checks: write
     pull-requests: write
   ```
   Establece permisos para el job. Permite leer contenido y problemas, y escribir en verificaciones y solicitudes de extracción.

3. **Entorno de Ejecución**:
   ```yaml
   runs-on: ubuntu-latest
   ```
   Especifica que el job se ejecutará en la última versión de Ubuntu.

4. **Pasos (Steps)**:
   Dentro de `steps`, se definen las acciones que se ejecutarán en este job.

   - **Checkout del Repositorio**:
     ```yaml
     - uses: actions/checkout@v3
     ```
     Clona el repositorio en el que se está ejecutando el workflow.

   - **Configurar JDK 11**:
     ```yaml
     - name: Set up JDK 11
       uses: actions/setup-java@v3
       with:
         java-version: '11'
         distribution: 'temurin'
         cache: maven
     ```
     Configura Java Development Kit (JDK) versión 11 usando la distribución de Temurin y habilita el cache para Maven.

   - **Compilación con Maven**:
     ```yaml
     - name: Build with Maven
       run: mvn clean package -B -Dmaven.test.skip
     ```
     Compila el proyecto usando Maven, omitiendo las pruebas.

   - **Preparar los Artefactos de Compilación**:
     ```yaml
     - run: mkdir candidate-binary && cp target/*.jar candidate-binary
     ```
     Crea un directorio llamado `candidate-binary` y copia los archivos JAR resultantes de la compilación en ese directorio.

   - **Ejecutar Pruebas con Maven**:
     ```yaml
     - name: Test with Maven
       run: mvn test
     ```
     Ejecuta las pruebas del proyecto utilizando Maven.

   - **Guardar Resultados de las Pruebas**:
     ```yaml
     - run: mkdir test-results && cp target/*-reports/TEST-*.xml test-results
     ```
     Crea un directorio `test-results` y copia los resultados de las pruebas (en formato XML) en ese directorio.

   - **Subir Artefactos**:
     ```yaml
     - uses: actions/upload-artifact@v3
       with:
         name: Application-Binary
         path: candidate-binary
     ```
     Sube los artefactos de compilación al almacenamiento de GitHub Actions.

     ```yaml
     - uses: actions/upload-artifact@v3
       with:
         name: Test-Results
         path: test-results
     ```
     Sube los resultados de las pruebas de la misma manera.

   - **Publicar Resultados de las Pruebas**:
     ```yaml
     - name: Publish Test Results
       uses: EnricoMi/publish-unit-test-result-action@v2.0.0
       if: always()
       with:
         junit_files: "test-results/**/*.xml"
     ```
     Publica los resultados de las pruebas en un formato legible, asegurando que se ejecute siempre, independientemente de si las pruebas pasaron o fallaron.

### Resumen
Este workflow configura un entorno Java, compila un proyecto Maven, ejecuta pruebas, y finalmente sube tanto el binario de la aplicación como los resultados de las pruebas a GitHub. Esto es útil para la integración continua, ya que permite automatizar el proceso de construcción y prueba del código.

