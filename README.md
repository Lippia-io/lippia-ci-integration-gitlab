# Gitlab pipeline example

 Es un proyecto que tiene como finalidad automatizar el testeo del codigo ingresado al repositorio, utilizando el framework Lippia.

## Consideraciones

El proyecto incluye la imagen de Lippia con todas las herramientas necesarias para los tests que se ejecutan segun donde se haga el commit o el merge:

- Cuando el commit se realiza a main o master el test se ejecuta automaticamente
- Cuando el commit se realiza a otro branch el test se debe ejecutar manualmente

## Como se usa

### CI dentro del repositorio de una app

En caso de ejecutar los test en el proceso de integración/delivery de una aplicación, se debe incluir el bloque ejemplo correspondiente.
Los cambios fundamentales a tener en cuenta son el clonado del repo dentro del before script:

```yaml
  ...
  before_script:
  ...
    - git clone https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.com/path/to/your/<AUTOMATION_PROJECT>.git"
    - cd <AUTOMATION_PROJECT>
  ...

```
Y las rules que permiten configurar que perfiles y test se van a ejecutar en cada ambiente dependiendo desde que branch se disparó el pipeline: 

```yaml
  
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'
      variables:
        TAG:  "@Regression"
        TYPE: "@Api"
        ENV:  "Dev"
        LANG: "@EN"
        # Your other configuration variables here
    - if: '$CI_COMMIT_BRANCH == "master"'
      variables:
        TAG:  "@Regression"
        TYPE: "@Api"
        ENV:  "Test"
        LANG: "@EN"
        # Your other configuration variables here
  ...

```

Para que el pipeline clone el repositorio donde se encuentre su proyecto de testing automatizado y ejecute dentro de la carpeta el test.

> Se recomienda configurar las credenciales en variables de entorno de la herramienta de CI para evitar filtrar contraseñas en el código del repositorio.

### CI dentro del repositorio de automation

Para poder ejecutar el pipeline se debe agregar el ejemplo correspondiente:

```yaml
Testing automation:
  stage: Test
  before_script:
    - |   
          echo "Valores de las variables"
          echo "TAG= $TAG"
          echo "TESTTYPE= $TESTTYPE"
          echo "LANG= $LANG"
        # Your other configuration variables here
  script:
    - | 
      mvn test -P$TESTTYPE -Dcrowdar.cucumber.filter="'$TAG'" -Dcrowdar.cucumber.filter.language="'$LANG'"

  rules:
    - if: '$CI_COMMIT_BRANCH == "master" || $CI_COMMIT_BRANCH == "main"'
      variables:
        TAG: "@Success"
        TESTTYPE: "Secuencial"
        LANG: "@EN"
        # Your other configuration variables here
    - if: '$CI_COMMIT_BRANCH != "master" && $CI_COMMIT_BRANCH != "main"'
      when: manual

  artifacts:
    when: always
    paths:
      - target/reports/

```

Este ejemplo es una simplificación que permite ejecutar las pruebas desde el repositorio que contiene el código de automation

* Un nuevo commit en el repositorio a las branches "main" o "master" dispara el pipeline automaticamente, iniciando las pruebas pertinentes. Si se desea cambiar esto se puede modificar en la siguiente sección del documento:

```
rules:
    - if: '$CI_COMMIT_BRANCH == "dev" || $CI_COMMIT_BRANCH == "test"'
     #se modifican las ramas, tambien puede eliminarse una o agregar otra
```
* Tambien se pueden modificar las ramas que disparan el pipeline de forma manual en la siguiente seccion del documento:

```
- if: '$CI_COMMIT_BRANCH != "dev" && $CI_COMMIT_BRANCH != "test"'
      when: manual
      #se modifican las ramas, tambien puede eliminarse una o agregar otra
```

* Este pipeline trabaja con la version de lippia 3.1.2.2, en caso de querer modificarla utilizar una imagen desde el siguiente link

> https://hub.docker.com/r/crowdar/lippia/tags


- Antes de disparar el pipeline se deben configurar las siguientes variables de entorno dentro del archivo .gitlab-ci.yml en la siguiente seccion:

```
variables:
        TAG: "@Smoke"
        TESTTYPE: parallel
        BROWSERTYPE: chromeHeadless
        LANG: "@EN"
```

- Los valores de dichas variables se encuentran en el archivo POM.xml

  * **TAG**: lleva el nombre de la prueba
  * **TESTTYPE**:  determina el tipo de pruebas a realizar
  * **BROWSERTYPE**: determina el perfil de buscador que se usara, para mas informacion ver documentacion lippia 
  * **LANG**: determina el idioma
  
**NOTA:  el pipeline permite modificar o agregar mas variables de entorno dentro del apartado "variables"**

* para realizar las pruebas utilizamos el comando: 
```
$ mvn clean test
```
* En caso de agregar o modificar variables de entorno realizar los cambios necesarios en el script del test en los archivos YAML

Los reportes son generados en una carpeta llamada **Target**, que sera generada una vez que la ejecucion de las pruebas haya finalizado.

* El artifact se encuentra para descargar en CI/CD --> Pipelines, dentro del pipeline finalizado, en la parte derecha de la pagina.


**para mas informacion ver [documentación lippia.](https://github.com/Crowdar/lippia-web-sample-project#getting-started "documentación lippia.")**
