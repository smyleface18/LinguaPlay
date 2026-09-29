---
titulo: Descripción del programa
subtitulo: Ficha técnica de LinguaPlay para el registro de soporte lógico ante la Dirección Nacional de Derecho de Autor
autores: "[PENDIENTE: nombres de los autores]"
institucion: "[PENDIENTE: institución]"
curso: "[PENDIENTE: programa o curso]"
docente: "[PENDIENTE: docente o asesor]"
fecha: "[PENDIENTE: fecha]"
version: "[PENDIENTE: versión del software]"
formato: ficha
---

# Descripción del programa

**Nombre del software:** LinguaPlay

**Tipo de obra:** Software (soporte lógico)

**Participación:** [PENDIENTE: descripción de la participación de cada autor]

**Autor(es):** [PENDIENTE: nombre completo, documento de identidad y datos de contacto de cada autor]

**Año:** [PENDIENTE: año de creación o de terminación de la obra]

## Nombre del software

LinguaPlay.

## Tipo de software

Aplicación multiplataforma (Android, iOS y web) con una API en la nube, orientada al aprendizaje y la práctica del inglés como lengua extranjera mediante juegos. Incluye partidas de preguntas por nivel del Marco Común Europeo de Referencia y la escritura colaborativa de historietas en inglés, con revisión automática del idioma, narración por voz sintética e ilustraciones generadas por inteligencia artificial.

## Arquitectura general

El sistema tiene una arquitectura cliente-servidor en tres componentes:

1. **Aplicación cliente (LP-MOB).** Desarrollada con React Native y Expo. Un solo código fuente genera las versiones para Android, iOS y navegador web. Se comunica con el servidor por HTTP (REST) y, para las partidas en tiempo real, por WebSocket (Socket.IO).
2. **Servidor de aplicación (LP-API).** Desarrollado con NestJS sobre Node.js y organizado en módulos funcionales. Centraliza las reglas de negocio, la autorización por roles, la coordinación de las partidas en tiempo real y la integración con los servicios externos.
3. **Persistencia e infraestructura.** Base de datos relacional PostgreSQL para la información permanente y Redis para el estado de las partidas en curso, los bloqueos distribuidos y las colas de tareas.

Los servicios externos son:

- AWS Cognito: identidad y autenticación.
- Amazon S3: almacenamiento de archivos.
- Amazon Bedrock: revisión del inglés y títulos.
- Amazon Polly: narración.
- Cloudflare Workers AI: ilustraciones.

El despliegue se hace con contenedores Docker.

## Módulos del sistema

- Usuarios y autenticación (registro, inicio de sesión, roles Jugador y Administrador).
- Perfil y estadísticas del jugador.
- Trivia en tiempo real (individual y multijugador).
- Historieta colaborativa en tiempo real (turnos, personajes y revisión del inglés con IA).
- Generación de media (narración por voz e ilustraciones de cada viñeta).
- Historial y catálogo de historietas, con reacciones por viñeta y likes.
- Administración de contenido: categorías, preguntas, opciones y archivos multimedia.
- Moderación de historietas (quitar, restaurar y regenerar imágenes, con historial).
- Dashboard de estadísticas de la aplicación.

## Flujo del sistema

1. **Acceso.** El usuario se registra o inicia sesión. El servidor valida su identidad con AWS Cognito y la aplicación muestra las funciones que corresponden a su rol.
2. **Trivia.** El jugador elige un nivel y juega sola o en una sala compartida por código. El servidor publica las preguntas con tiempo límite, valida las respuestas, calcula el puntaje según la rapidez y guarda los resultados.
3. **Historieta.** Los jugadores escriben por turnos las viñetas de una historieta. Cada borrador lo revisa la IA, que explica los errores en español. Al terminar, el sistema narra e ilustra cada viñeta, le pone un título y la publica.
4. **Catálogo.** Cualquier usuario puede leer las historietas publicadas, reaccionar a sus viñetas y darles like.
5. **Administración.** El administrador gestiona el contenido de la trivia, consulta las estadísticas y modera las historietas publicadas.

## Modelo de datos

Base de datos relacional (PostgreSQL) con las entidades:

- **Usuarios:** usuario.
- **Contenido de la trivia:** categoría de preguntas, pregunta, opción de respuesta y archivo multimedia.
- **Resultados de la trivia:** partida, sesión de juego y respuesta del jugador.
- **Historietas:** historieta, viñeta, participante, like e historial de moderación.

Las relaciones principales son:

- una categoría agrupa preguntas y cada pregunta tiene opciones;
- una partida registra la sesión y las respuestas de cada jugador;
- una historieta se compone de viñetas y tiene participantes, likes y un historial de moderación.

El estado temporal de las partidas en curso se guarda en Redis.

## Tecnologías utilizadas

- **Lenguaje:** TypeScript.
- **Cliente:** React Native, Expo, React Navigation y Zustand.
- **Servidor:** Node.js, NestJS, TypeORM, Socket.IO y BullMQ.
- **Datos:** PostgreSQL y Redis.
- **Servicios de nube:** AWS Cognito, Amazon S3, Amazon Bedrock (Nova 2 Lite), Amazon Polly y Cloudflare Workers AI (FLUX.1 schnell).
- **Infraestructura:** Docker y nginx.

## Participación en el desarrollo

[PENDIENTE: por cada autor, su nombre y los componentes o módulos que desarrolló. Por ejemplo: "La participación de [nombre] corresponde a …".]

[PENDIENTE: indicar si la obra es individual, en colaboración o colectiva, y si hay un titular de derechos patrimoniales distinto de los autores, como una institución o empresa.]
