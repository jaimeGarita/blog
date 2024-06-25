---
    title: "Dockerfile"
    description: "En este post se va a hablar sobre todo lo necesario bajo mi opinión de dockerfile y sus instrucciones"
    publishDate: "25 Jun 2024"
    updatedDate: 25 Jun 2024
    tags: ["dockerfile", "microservicios"]
---

## ¿Qué es Dockerfile?

### Según microsoft

> dockerfile es un archivo de texto que contiene las instrucciones necesarias para crear una nueva imágen en el contenedor[^1]

[^1]: Un contenedor es como una caja autónoma que contiene todo lo necesario para hacer funcionar tu aplicación, haciendo que sea fácil de mover y ejecutar en diferentes lugares

### Según mi experiencia

Dockerfile no es más que un fichero al que tienes que llamarle Dockerfile _(literalmente)_. Dentro de este fichero tú vas a tener que añadir una serie de instrucciones para que tu proyecto pueda funcionar. Esto te va a crear un elemento llamado Imágen[^2].

[^2]: Imagínate que una imagen en Docker es como un paquete con todo lo necesario para que tu aplicación funcione sin ningún problema. Desde el código hasta las herramientas más necesarias.

### Pongamos un pequeño ejemplo

Todos los proyectos Java necesitan su ración de Java, ¿no? Pues aquí no es la excepción. Necesitamos un buen _OpenJDK_ y un archivo _.jar_, que es básicamente el fruto de nuestras horas de compilación, donde residen todas esas clases .class que tanto nos ha costado escribir.

#### ¿cómo metemos todo en nuestro Dockerfile?

parece fácil ¿verdad? Pero... ¿será tan sencillo como parece?

Bajo mi opinión, todos debemos de empezar creando un Dockerfile más _sencillo_ como el ejemplo que se va a poner a continuación:

```dockerfile

    FROM openjdk:11

    WORKDIR /app

    COPY MiApp.jar /app/MiApp.jar

    CMD ["java", "-jar", "MiApp.jar"]

```

| Opción      | Descripción                                                                                                                                                                                                                                                  |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **FROM**    | Esta línea muestra la imagen que se va a usar en el contenedor.                                                                                                                                                                                              |
| **WORKDIR** | Se establece en donde se va a trabajar, posteriormente se van a usar para las instrucciones en el contenedor.                                                                                                                                                |
| **COPY**    | Como esta instrucción indica, nos permite copiar uno o varios archivos de nuestro local a la imagen.                                                                                                                                                         |
| **CMD**     | Define el ejecutable por defecto de una imagen Docker. En este caso, el contenedor ejecuta el proceso especificado por el terminal. Para que sea más sencillo, todas las instrucciones necesarias que tú pondrías en una terminal para ejecutar el proyecto. |

---

Este fichero dockerfile es el que vas a encontrar en el 90% de las ocasiones, sobre todo cuando el proyecto se esta inicializando y la imágen no necesita más como para ser ejecutada en un contenedor.

---

#### Opción mas compleja

Ahora vamos a ver un fichero Dockerfile con instrucciones que en mi caso, menos veces he visto:

```dockerfile

FROM openjdk:18-jdk-alpine as builder

WORKDIR /app/usermanagement_ms

COPY . .

COPY ./.mvn ./.mvn

COPY ./mvnw .

COPY ./pom.xml .

RUN ./mvnw clean package && rm -r ./target

COPY ./src ./src

RUN ./mvnw clean package

FROM openjdk:18-jdk-alpine

WORKDIR /app

COPY  --from=builder /app/usermanagement_ms/target/usermanagement-0.0.1-SNAPSHOT.jar .

EXPOSE 8001

ENTRYPOINT ["java","-jar","./target/usermanagement-0.0.1-SNAPSHOT.jar"]

```

_Primera etapa_

| Opción                         | Descripción                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------- |
| **FROM**                       | Esta línea muestra la imagen que se va a usar en el contenedor.                       |
| **COPY .**                     | Copia todos los archivos del proyecto al directorio de trabajo dentro del contenedor. |
| **COPY ./.mvn ./.mvn**         | Copia el directorio .mvn que contiene el wrapper de Maven.                            |
| **COPY ./mvnw .**              | Copia el ejecutable del wrapper de Maven.                                             |
| **COPY ./pom.xml .**           | Copia el archivo pom.xml que contiene la configuración de Maven.                      |
| **COPY ./pom.xml .**           | Copia el archivo pom.xml que contiene la configuración de Maven.                      |
| **COPY ./pom.xml .**           | Copia el archivo pom.xml que contiene la configuración de Maven.                      |
| **RUN ./mvnw clean package .** | Ejecuta Maven para limpiar, compilar y empaquetar el proyecto.                        |
| **COPY ./src ./src**           | Copia el código fuente al directorio src después de la compilación.                   |

_Segunda etapa_

| Opción                         | Descripción                                                                                                                         |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- |
| **FROM**                       | Esta línea muestra la imagen que se va a usar en el contenedor.                                                                     |
| **WORKDIR /app**               | Establece el directorio de trabajo (WORKDIR) en /app                                                                                |
| **COPY --from=builder ...**    | Copia el archivo JAR generado (usermanagement-0.0.1-SNAPSHOT.jar) desde la primera etapa (builder) al directorio de trabajo (/app). |
| **EXPOSE 8001**                | Expone el puerto 8001, si tu aplicación Spring Boot está configurada para escuchar en ese puerto (ajusta según sea necesario).      |
| **ENTRYPOINT**                 | Define el comando de inicio para ejecutar la aplicación Spring Boot al iniciar el contenedor.                                       |
| **COPY ./pom.xml .**           | Copia el archivo pom.xml que contiene la configuración de Maven.                                                                    |
| **COPY ./pom.xml .**           | Copia el archivo pom.xml que contiene la configuración de Maven.                                                                    |
| **RUN ./mvnw clean package .** | Ejecuta Maven para limpiar, compilar y empaquetar el proyecto.                                                                      |
| **COPY ./src ./src**           | Copia el código fuente al directorio src después de la compilación.                                                                 |

---

Te has quedado de piedra, como yo la primera vez cuando empecé a investigar hasta que nivel pueden llegar los ficheros Dockerfile (tranquilo a todo el mundo le pasa)
Ahora bien, te voy a explicar que hemos implementado aquí y obviamente te voy a razonar todo _(bajo mi experiencia)_

---

1. **Optimización del tamaño de la imágen** Esto podemos separarlo en dos etápas:

   - La primer etapa es la etapa **Builder**, esta etapa contiene las herramientas de compilación permitiendo empaquetar la aplicación sin llevarnos el código.
   - La segunda etapa se usa una imágen mas ligera **openjdk:18-jdk-alpine** que solo contiene el entorno de ejecución de Java 18 en alpine Linux. Reduciendo el tamaño de la imágen final.

2. **Separación de fases:**

   - Gracias a separarlo hemos mejorado la seguridad al eliminar las herramientas de compilación _(rm -r ./target)_

3. **Mantenimiento:**
   - Ahora imagina que tu microservicio crece un 70% más de lo que tenias hace 3 meses de desarrollo, gracias a este Dockerfile conseguimos una mejora drástica en el mantenimiento separando las responsabilidades de cada etápa

### ¿Cual elegir?

Bajo mi punto de vista las dos opciones son correctas y como dije anteriormente, se suele usar las opciones mas fáciles ya que no necesitas tanta complejidad en los Dockerfile. Aun así creo que es importante y necesario saber todo el potencial que tiene Dockerfile.

---
    
Espero que esta introducción a los Dockerfile y microservicios haya sido útil y te haya inspirado a explorar nuevas formas de desarrollar y desplegar aplicaciones!

---

Esto es solo un pequeño ejemplo pero como todo, se puede hacer mucho mas complejo
