# Documentación técnica de LinguaPlay

## 1. Introducción

LinguaPlay es una plataforma de aprendizaje y entretenimiento basada en preguntas de idiomas. Permite gestionar usuarios, categorías, preguntas y opciones de respuesta, además de crear partidas multijugador en tiempo real.

El sistema está dividido en tres componentes principales:

- **LP-MOB:** cliente desarrollado con React Native y Expo para Android, iOS y web.
- **LP-API:** API desarrollada con NestJS que expone servicios HTTP y comunicación WebSocket.
- **linguaplay-core:** librería compartida con tipos, constantes y contratos utilizados por los demás componentes.

Este documento describe la arquitectura, tecnologías, estructura, datos, servicios, instalación, pruebas, despliegue y mantenimiento del sistema.

## 2. Descripción del sistema

LinguaPlay utiliza una arquitectura cliente-servidor. El cliente permite autenticarse, consultar el contenido educativo y participar en partidas. La API centraliza las reglas de negocio, la autorización, la persistencia y la coordinación de partidas.

Las funciones principales son:

- Registro e inicio de sesión de usuarios.
- Gestión de perfiles y roles.
- Gestión de categorías de preguntas.
- Gestión de preguntas y opciones de respuesta.
- Creación y unión a salas de juego.
- Partidas multijugador en tiempo real.
- Validación de respuestas y actualización de puntuaciones.
- Persistencia de sesiones, respuestas y resultados.
- Administración de sesiones mediante tokens de AWS Cognito.

La comunicación convencional se realiza mediante HTTP REST. Las partidas utilizan Socket.IO sobre WebSocket, dentro del namespace `/game`.

## 3. Requisitos

### 3.1 Requisitos de usuario

- Dispositivo Android, iOS o navegador web moderno.
- Conexión estable a Internet o a la red donde esté publicada la API.
- Acceso a los servicios de autenticación de AWS Cognito.
- Audio habilitado cuando se utilicen contenidos multimedia.

### 3.2 Requisitos de desarrollo

- Git con soporte para submódulos.
- Node.js 20 LTS o superior recomendado.
- Yarn.
- Docker Desktop.
- Android Studio y Android SDK para Android.
- macOS, Xcode y CocoaPods para compilación nativa de iOS.
- Navegador moderno para ejecutar Expo Web.

El repositorio no fija una versión exacta de Node.js ni valores mínimos oficiales de CPU o memoria. Los valores anteriores son recomendaciones para desarrollo y operación.

### 3.3 Requisitos de infraestructura

- PostgreSQL 15.
- Redis Stack Server.
- AWS Cognito configurado con región, User Pool y Client ID.
- Servidor capaz de aceptar tráfico HTTP y WebSocket.
- Puertos disponibles, por defecto:
  - API: `3000`.
  - PostgreSQL: `5432`.
  - Redis: `6379`.
  - Redis Commander: `8081`.

## 4. Arquitectura del sistema

### 4.1 Vista general

El cliente LP-MOB consume la API mediante HTTP y mantiene una conexión Socket.IO para las partidas. LP-API utiliza PostgreSQL para los datos persistentes, Redis para el estado temporal y la coordinación de conexiones, BullMQ para trabajos programados y AWS Cognito para la identidad de los usuarios.

```text
+-----------------------+
| LP-MOB                 |
| React Native + Expo   |
+-----------+-----------+
            |
       HTTP REST
            |
       Socket.IO /game
            |
+-----------v-----------+
| LP-API                |
| NestJS                |
+---+--------+-----+----+
    |        |     |
    |        |     +--> AWS Cognito
    |        +--------> Redis
    +-----------------> PostgreSQL
             |
             +-------> BullMQ
```

### 4.2 Módulos del backend

Los módulos principales registrados en `AppModule` son:

- `AuthModule`: registro, autenticación, tokens y usuario actual.
- `DatabaseModule`: conexión TypeORM y entidades.
- `QuestionModule`: operaciones sobre preguntas.
- `QuestionOptionsModule`: operaciones sobre opciones.
- `CategoryQuestionModule`: operaciones sobre categorías.
- `GameModule`: gateway y lógica de partidas.
- `MatchModule`: creación, unión y estado de partidas.
- `GameQueueModule`: trabajos temporizados de las partidas.
- `WsAuthModule`: validación de tokens en conexiones WebSocket.
- `CommonModule`: configuración, respuestas, filtros y utilidades compartidas.

### 4.3 Flujo de autenticación

1. LP-MOB envía las credenciales a `POST /auth/signIn`.
2. LP-API solicita la autenticación a AWS Cognito.
3. Cognito devuelve los tokens de acceso, identidad y renovación.
4. La aplicación conserva la sesión y utiliza el token en las solicitudes protegidas.
5. Para Socket.IO, el token se envía durante el handshake.
6. El gateway valida el token antes de aceptar la conexión de juego.

### 4.4 Flujo de una partida

1. El usuario crea una partida o introduce el identificador de una sala.
2. El cliente emite `createGame` o `joinGame` al namespace `/game`.
3. El backend crea o recupera el estado de la partida.
4. Redis conserva el estado temporal y los jugadores conectados.
5. El propietario inicia la partida mediante `startGame`.
6. BullMQ programa los tiempos de inicio y cambio de pregunta.
7. Los clientes reciben eventos de preguntas, respuestas, jugadores y resultados.

## 5. Tecnologías utilizadas

| Área         | Tecnología                        | Uso                                   |
| ------------ | --------------------------------- | ------------------------------------- |
| Cliente      | React Native `0.81.5`             | Interfaz multiplataforma              |
| Cliente      | Expo `~54.0.23`                   | Ejecución y herramientas del cliente  |
| Cliente      | React `19.1.0`                    | Componentes y estado de interfaz      |
| Cliente      | React Navigation `7`              | Navegación                            |
| Cliente      | Zustand `5`                       | Estado global                         |
| Cliente      | Socket.IO Client `4.8.1`          | Partidas en tiempo real               |
| Cliente      | Expo Audio / Expo Video           | Audio y vídeo                         |
| Backend      | NestJS `11`                       | API y módulos del servidor            |
| Backend      | TypeScript                        | Lenguaje principal                    |
| Backend      | Socket.IO `4.8.1`                 | Comunicación bidireccional            |
| Persistencia | PostgreSQL `15`                   | Base de datos relacional              |
| Persistencia | TypeORM `0.3.27`                  | Mapeo objeto-relacional y migraciones |
| Caché        | Redis                             | Estado temporal y coordinación        |
| Colas        | BullMQ                            | Trabajos programados de partidas      |
| Identidad    | AWS Cognito                       | Registro y autenticación              |
| Contenedores | Docker Compose                    | PostgreSQL y Redis en desarrollo      |
| Calidad      | Jest, Supertest, ESLint, Prettier | Pruebas y análisis del código         |

## 6. Estructura del proyecto

```text
LinguaPlay/
├── apps/
│   ├── LP-API/
│   │   ├── src/
│   │   │   ├── common/
│   │   │   ├── db/
│   │   │   │   ├── entities/
│   │   │   │   └── migrations/
│   │   │   └── modules/
│   │   │       ├── auth/
│   │   │       ├── category-question/
│   │   │       ├── game/
│   │   │       ├── question/
│   │   │       └── question-options/
│   │   ├── test/
│   │   ├── docker-compose.yml
│   │   └── package.json
│   └── LP-MOB/
│       ├── src/
│       │   ├── app/
│       │   ├── assets/
│       │   ├── features/
│       │   │   ├── auth/
│       │   │   ├── category/
│       │   │   ├── game/
│       │   │   └── question/
│       │   ├── shared/
│       │   └── store/
│       ├── app.json
│       └── package.json
├── linguaplay-core/
│   └── src/
│       ├── api/
│       ├── constants/
│       └── types/
└── docs/
    └── diagrams/
```

Cada aplicación y `linguaplay-core` se gestionan como submódulos Git independientes.

## 7. Diseño de la base de datos

### 7.1 Motor y configuración

La base de datos utiliza PostgreSQL y se accede mediante TypeORM. La opción `synchronize` está desactivada y las migraciones no se ejecutan automáticamente. Por tanto, los cambios de esquema deben aplicarse de forma controlada mediante los comandos de migración.

### 7.2 Entidades principales

- **User:** usuario, correo, nombre, rol, nivel, puntuación y avatar.
- **Game:** configuración general de un juego y su dificultad.
- **GameSession:** participación de un usuario en un juego, puntuación y posición.
- **CategoryQuestion:** categoría, nivel, descripción y tipo.
- **Question:** contenido, información adicional, categoría y límite de tiempo.
- **QuestionOption:** contenido de una opción y marca de respuesta correcta.
- **PlayerAnswer:** respuesta seleccionada, corrección y tiempo empleado.
- **GameQuestion:** relación entre juegos y preguntas.

### 7.3 Relaciones

- Un usuario puede tener varias sesiones de juego.
- Un juego puede contener varias sesiones.
- Una categoría puede agrupar varias preguntas.
- Una pregunta puede tener varias opciones.
- Una sesión puede registrar varias respuestas.
- Una pregunta puede estar asociada a varios juegos.
- Una opción puede ser seleccionada en varias respuestas.

El modelo relacional visual se encuentra en `docs/diagrams/relational-model.puml`.

## 8. API / Servicios

### 8.1 API REST

La API se ejecuta por defecto en el puerto `3000`. Los endpoints identificados son:

| Método | Ruta                      | Descripción                    |
| ------ | ------------------------- | ------------------------------ |
| POST   | `/auth/signUp`            | Registrar usuario              |
| POST   | `/auth/signIn`            | Iniciar sesión                 |
| POST   | `/auth/refresh-token`     | Renovar tokens                 |
| GET    | `/auth/me`                | Obtener el usuario autenticado |
| POST   | `/auth/revoke-token`      | Revocar token de renovación    |
| POST   | `/category-question`      | Crear categoría                |
| GET    | `/category-question`      | Listar categorías              |
| GET    | `/category-question/:id`  | Consultar categoría            |
| PATCH  | `/category-question/:id`  | Actualizar categoría           |
| DELETE | `/category-question/:id`  | Eliminar categoría             |
| POST   | `/question`               | Crear pregunta                 |
| POST   | `/question/batch`         | Crear preguntas en lote        |
| GET    | `/question`               | Listar preguntas               |
| GET    | `/question/:id`           | Consultar pregunta             |
| PATCH  | `/question/:id`           | Actualizar pregunta            |
| DELETE | `/question/:id`           | Eliminar pregunta              |
| POST   | `/question-options`       | Crear opción                   |
| POST   | `/question-options/batch` | Crear opciones en lote         |
| GET    | `/question-options`       | Listar opciones                |
| GET    | `/question-options/:id`   | Consultar opción               |
| PATCH  | `/question-options/:id`   | Actualizar opción              |
| DELETE | `/question-options/:id`   | Eliminar opción                |

Las rutas de usuario actual y revocación requieren autenticación JWT. Las operaciones administrativas también dependen del rol del usuario.

### 8.2 Eventos WebSocket

El gateway utiliza el namespace `/game` y valida el token recibido en `handshake.auth.token`.

| Evento           | Dirección           | Función                              |
| ---------------- | ------------------- | ------------------------------------ |
| `createGame`     | Cliente → servidor  | Crear una sala                       |
| `joinGame`       | Cliente → servidor  | Unirse a una sala                    |
| `startGame`      | Cliente → servidor  | Iniciar la partida                   |
| `answer`         | Cliente → servidor  | Enviar una respuesta                 |
| `requestRematch` | Cliente → servidor  | Solicitar revancha                   |
| `leaveRoom`      | Cliente → servidor  | Abandonar la sala                    |
| `playersUpdated` | Servidor → clientes | Actualizar jugadores y puntuaciones  |
| `newQuestion`    | Servidor → clientes | Publicar una pregunta                |
| `answerResult`   | Servidor → cliente  | Informar si la respuesta es correcta |
| `questionEnded`  | Servidor → clientes | Informar el fin de una pregunta      |
| `gameEnded`      | Servidor → clientes | Informar el fin de la partida        |

### 8.3 Respuestas y validación

La API utiliza un formato de respuesta común mediante un interceptor global. También usa `ValidationPipe` con lista blanca de propiedades, rechazo de propiedades no permitidas y conversión implícita de tipos.

## 10. Instalación y configuración

### 10.1 Descargar el repositorio

```bash
git clone --recurse-submodules https://github.com/smyleface18/LinguaPlay.git
cd LinguaPlay
```

Si ya se clonó sin submódulos:

```bash
git submodule update --init --recursive
```

### 10.2 Instalar dependencias

```bash
cd apps/LP-API
yarn install

cd ../LP-MOB
yarn install

cd ../../linguaplay-core
yarn install
yarn build
```

### 10.3 Configurar variables de entorno

Crear `apps/LP-API/.env` a partir de `.env.template`:

```env
DB_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_USER=admin
POSTGRES_PASSWORD=***
POSTGRES_DB=LGP-DB
PORT=3000
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=redis://localhost:6379
MATCH_TTL=3600
AWS_REGION=***
COGNITO_USER_POOL_ID=***
COGNITO_CLIENT_ID=***
```

### 10.4 Iniciar infraestructura y API

```bash
cd apps/LP-API
docker compose up -d
yarn migration:run
yarn start:dev
```

### 10.5 Configurar y ejecutar el cliente

La URL del backend se define en `apps/LP-MOB/src/shared/api/apiConfig.ts`. En un dispositivo físico se debe sustituir `localhost` por la IP accesible del servidor.

```bash
cd apps/LP-MOB
yarn start
```

Comandos alternativos:

```bash
yarn android
yarn ios
yarn web
```

## 11. Pruebas

### 11.1 Pruebas del backend

Desde `apps/LP-API`:

```bash
yarn test
yarn test:e2e
yarn test:cov
```

También están disponibles los modos de observación y depuración:

```bash
yarn test:watch
yarn test:debug
```

La configuración de Jest utiliza `ts-jest` y un entorno Node.js. Las pruebas e2e usan Supertest y comprueban la inicialización de la aplicación y la respuesta de la ruta raíz.

### 11.2 Validaciones recomendadas

- Comprobar que la API inicia sin errores.
- Verificar la conexión con PostgreSQL y Redis.
- Ejecutar las migraciones en una base de datos de prueba.
- Probar registro, inicio de sesión, renovación y revocación de tokens.
- Verificar autorización por rol.
- Crear y unirse a una partida desde dos clientes.
- Validar el envío de respuestas, cambio de preguntas y finalización de la partida.
- Ejecutar lint y compilación antes de desplegar.

## 12. Despliegue

### 12.1 Backend

El backend puede compilarse y ejecutarse con:

```bash
cd apps/LP-API
yarn install
yarn build
yarn migration:run
yarn start:prod
```

El servidor debe escuchar en una interfaz accesible para los clientes y publicar el puerto de la API. En producción se recomienda utilizar un proxy inverso con HTTPS y soporte para WebSocket.

### 12.2 Servicios de datos

PostgreSQL y Redis deben ejecutarse como servicios persistentes. PostgreSQL requiere un volumen de datos y Redis debe estar protegido mediante red privada, autenticación y reglas de firewall adecuadas.

### 12.3 Cliente

Para distribución móvil se deben generar los artefactos correspondientes de Expo/EAS o del entorno nativo configurado. Para web se puede generar la versión web de Expo y publicarla en un servidor compatible.

Antes de publicar el cliente, debe sustituirse la URL local de la API por la URL pública del entorno correspondiente.

### 12.4 Configuración de producción

- Usar HTTPS para HTTP y WebSocket.
- Configurar CORS con orígenes concretos.
- Mantener las credenciales fuera del repositorio.
- Configurar correctamente `AWS_REGION`, `COGNITO_USER_POOL_ID` y `COGNITO_CLIENT_ID`.
- Ejecutar migraciones antes de iniciar una versión que requiera cambios de esquema.
- Configurar logs, copias de seguridad y supervisión de PostgreSQL y Redis.

## 13. Mantenimiento

### 13.1 Código y dependencias

- Mantener actualizados los submódulos y registrar sus referencias en el repositorio principal.
- Instalar dependencias con Yarn usando los archivos `yarn.lock`.
- Revisar cambios incompatibles de Expo, React Native, NestJS y Socket.IO antes de actualizar.
- Ejecutar compilación, pruebas y lint después de cada actualización relevante.

### 13.2 Base de datos

- Realizar copias de seguridad periódicas de PostgreSQL.
- Revisar las migraciones antes de aplicarlas en producción.
- No activar `synchronize` en producción.
- Verificar índices, crecimiento de tablas y registros históricos.

### 13.3 Redis y partidas

- Supervisar memoria y conexiones de Redis.
- Revisar el valor de `MATCH_TTL` cuando cambien las reglas de expiración de partidas.
- Comprobar que BullMQ procese los trabajos temporizados.
- Limpiar estados temporales que queden después de errores o desconexiones.

### 13.4 Seguridad y operación

- Rotar credenciales y secretos periódicamente.
- No almacenar contraseñas ni tokens en logs.
- Revisar los permisos de AWS Cognito y los roles de usuario.
- Limitar los puertos públicos al mínimo necesario.
- Supervisar errores de autenticación, conexiones WebSocket y fallos de base de datos.
- Mantener actualizado el certificado HTTPS.

### 13.5 Documentación complementaria

Los diagramas PlantUML disponibles en `docs/diagrams/` complementan este documento:

- `component-diagram.puml`: componentes y dependencias.
- `deployment-diagram.puml`: nodos de despliegue.
- `relational-model.puml`: modelo relacional.
- `sequence-auth.puml`: autenticación.
- `sequence-game-flow.puml`: flujo de partida.
- `class-diagram.puml`: clases y entidades.
- `use-case.puml`: casos de uso.

## Nota sobre las interfaces del sistema

La sección de interfaces visuales no se incluye en este documento. Las capturas de pantalla y la explicación de cada vista se incorporarán posteriormente en el apartado correspondiente del manual.
