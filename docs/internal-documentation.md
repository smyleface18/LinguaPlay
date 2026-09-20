# Documentación interna de LinguaPlay

## 1. Propósito y alcance

LinguaPlay es una plataforma de aprendizaje y entretenimiento basada en preguntas de idiomas. Permite registrar usuarios, gestionar categorías, preguntas y opciones de respuesta, y participar en partidas multijugador en tiempo real.

El proyecto está compuesto por:

- `apps/LP-MOB`: cliente React Native con Expo para Android, iOS y web.
- `apps/LP-API`: API NestJS con servicios HTTP y WebSocket.
- `linguaplay-core`: tipos, constantes y contratos compartidos.

Esta documentación está orientada al equipo. Explica cómo incorporarse, desarrollar, revisar y mantener el proyecto. La arquitectura, tecnologías, modelo de datos, API completa, instalación, pruebas y despliegue están descritos en [technical-documentation.md](technical-documentation.md).

### Usuarios y roles

- `PLAYER`: usuario que se autentica y participa en partidas.
- `ADMIN`: usuario con acceso a las funciones administrativas disponibles.

Un registro nuevo se asigna al grupo `PLAYER` de AWS Cognito. El cliente selecciona `AdminStack` para usuarios con rol `ADMIN` y `UserStack` para el resto; la autorización definitiva debe comprobarse siempre en el backend.

## 2. Incorporación de desarrolladores

### Primer día

1. Instalar Git, Node.js 20 LTS o superior, Yarn y Docker Desktop.
2. Clonar el repositorio con sus submódulos:

   ```bash
   git clone --recurse-submodules https://github.com/smyleface18/LinguaPlay.git
   cd LinguaPlay
   ```

3. Si ya se clonó sin submódulos:

   ```bash
   git submodule update --init --recursive
   ```

4. Instalar dependencias:

   ```bash
   cd apps/LP-API && yarn install
   cd ../LP-MOB && yarn install
   cd ../../linguaplay-core && yarn install && yarn build
   ```

5. Crear `apps/LP-API/.env` a partir de `.env.template`. Solicitar las credenciales de AWS Cognito al responsable del entorno.
6. Iniciar infraestructura y API:

   ```bash
   cd apps/LP-API
   docker compose up -d
   yarn migration:run
   yarn start:dev
   ```

7. Revisar `apps/LP-MOB/src/shared/api/apiConfig.ts`. En un teléfono físico, reemplazar `localhost` por la IP accesible del equipo que ejecuta la API.
8. Iniciar el cliente:

   ```bash
   cd apps/LP-MOB
   yarn start
   ```

### Ubicación de cada componente

- Módulos de negocio backend: `apps/LP-API/src/modules/`.
- Entidades y migraciones: `apps/LP-API/src/db/`.
- Utilidades y configuración backend: `apps/LP-API/src/common/`.
- Features y pantallas: `apps/LP-MOB/src/features/`.
- Navegación: `apps/LP-MOB/src/app/navigation/`.
- Cliente HTTP: `apps/LP-MOB/src/shared/api/`.
- Estado global: `apps/LP-MOB/src/store/`.
- Tipos compartidos: `linguaplay-core/src/`.
- Diagramas: `docs/diagrams/`.

## 3. Configuración y comandos

### Variables de entorno

Las variables principales de `apps/LP-API/.env` son:

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

Los valores de AWS Cognito son específicos de cada entorno. No publicar secretos ni reutilizar credenciales sin autorización.

### Comandos de API

Desde `apps/LP-API`:

```bash
yarn start:dev
yarn build
yarn lint
yarn test
yarn test:e2e
yarn test:cov
yarn migration:show
yarn migration:run
yarn migration:revert
```

El script `lint` ejecuta ESLint con `--fix`; revisar el diff después de usarlo.

### Comandos del cliente

Desde `apps/LP-MOB`:

```bash
yarn start
yarn android
yarn ios
yarn web
```

El paquete móvil no define actualmente scripts de pruebas o build propios en `package.json`.

### Comandos del core

Desde `linguaplay-core`:

```bash
yarn build
yarn lint
yarn format
```

Después de modificar el core, compilarlo antes de probar sus consumidores.

## 4. Flujo de desarrollo

1. Identificar el componente propietario: API, cliente o core.
2. Revisar la documentación técnica y buscar una implementación existente similar.
3. Confirmar si el cambio afecta contratos compartidos, migraciones, autenticación o eventos WebSocket.
4. Modificar primero `linguaplay-core` si cambia un tipo, ruta o constante compartida.
5. En la API, actualizar DTO, servicio, controlador o gateway, persistencia y autorización según corresponda.
6. Crear una migración explícita si cambia el esquema; no activar `synchronize`.
7. En el cliente, actualizar el servicio, hook, estado y navegación afectados.
8. Ejecutar build, lint y pruebas del componente afectado.
9. Probar el recorrido completo cuando intervengan autenticación, partidas o más de un componente.
10. Actualizar esta documentación o la documentación técnica si cambia el contrato o el flujo operativo.

### Cambios de base de datos

- Revisar primero las entidades existentes.
- Crear y revisar una migración.
- Aplicarla en desarrollo antes de ejecutar las pruebas.
- Incluir la migración en el mismo cambio que necesita el nuevo esquema.
- No modificar datos de producción manualmente sin registrar el procedimiento.

### Cambios de partidas

La lógica de estado está en `apps/LP-API/src/modules/game/match/domain/match.entity.ts`; la coordinación está en `match.service.ts` y `game.gateway.ts`. Un cambio en estados, puntuación, tiempos, reconexión o payloads requiere revisar también `SocketEvents.ts`, `socket.service.ts` y `useGame` en el cliente.

## 5. Convenciones y buenas prácticas observadas

- El código se organiza por features y módulos de dominio.
- La API usa servicios, controladores, DTOs, guards y entidades TypeORM.
- El cliente separa pantallas, hooks, servicios, componentes compartidos y estado.
- Se utilizan alias de importación como `@/` en el cliente y backend.
- Las validaciones se realizan mediante DTOs y `ValidationPipe` en la API, además de validaciones inmediatas en formularios del cliente.
- Los estados temporales de partidas se serializan en Redis; los cambios de `Match` deben mantener compatibilidad con `toPersistence` y `fromPersistence`.
- Los nombres de eventos Socket.IO son parte del contrato entre cliente y servidor.
- No se deben guardar secretos, tokens ni credenciales en el repositorio o en logs.
- Antes de crear una abstracción nueva, buscar el patrón equivalente ya usado en el componente.

No existe un documento que establezca una convención formal completa de nombres. Mantener el estilo del módulo vecino y usar nombres descriptivos.

## 6. Flujo de trabajo con Git

### Repositorios y submódulos

El repositorio principal referencia:

- `apps/LP-API` → `https://github.com/smyleface18/LP-API.git`
- `apps/LP-MOB` → `https://github.com/smyleface18/LP-MOB.git`
- `linguaplay-core` → `https://github.com/smyleface18/linguaplay-core.git`

Cada submódulo tiene historial propio. Un cambio que cruza repositorios requiere publicar primero el commit del submódulo y después actualizar su referencia en el repositorio principal.

### Ramas y commits

El historial revisado muestra `main` como rama activa en el repositorio principal y submódulos. No existe una estrategia formal documentada para `develop`, releases, hotfixes o nombres obligatorios de ramas.

Se observan prefijos de commit como `feat`, `fix`, `refactor`, `chore` y `ci`, aunque también hay mensajes con formatos diferentes. Es una práctica observada, no un estándar obligatorio. Un formato recomendado compatible con el historial es:

```text
<tipo>(<ámbito>): descripción breve
```

### Cambiar un submódulo

```bash
cd apps/LP-API
git status
git add src test
git commit -m "feat(auth): adjust sign-in flow"
git push

cd ../..
git status
git add apps/LP-API
git commit -m "chore: update LP-API submodule"
git push
```

No hacer commit en el repositorio principal intentando incluir los archivos internos del submódulo; primero debe existir el commit del submódulo.

### Pull Requests y revisión

No se encontraron plantillas de Pull Request, reglas de aprobación ni proceso formal de Code Review en el repositorio. Antes de integrar, se recomienda revisar el diff, ejecutar build y pruebas, comprobar migraciones y contratos, verificar que no haya secretos y confirmar la referencia del submódulo.

Para sincronizar sin mover submódulos a referencias no revisadas:

```bash
git pull
git submodule update --init --recursive
```

## 7. Flujos funcionales

### Registro e inicio de sesión

```text
Usuario → Signup/SignIn → AuthService del cliente → API REST
→ AWS Cognito + UserRepository → tokens → store y almacenamiento local
→ AppNavigator
```

El registro comprueba el correo, crea la identidad en Cognito, asigna `PLAYER` y guarda el usuario local. El inicio de sesión devuelve tokens. La restauración busca `accessToken`; la renovación usa `refreshToken`.

### Gestión de contenido

```text
Administrador → pantalla → hook → servicio HTTP → controlador API
→ servicio de dominio → TypeORM/PostgreSQL → respuesta → pantalla
```

Revisar siempre las validaciones del formulario y del DTO. Los controladores de categorías, preguntas y opciones no tienen exactamente las mismas protecciones en el código actual; comprobar el controlador concreto antes de asumir una regla de acceso.

### Partida en tiempo real

```text
Jugador → GameScreen → SocketService → Socket.IO /game
→ GameGateway → MatchService → Redis
→ eventos de sala → clientes
```

`createGame` crea una sala y `joinGame` incorpora un jugador. El propietario inicia con `startGame`. BullMQ coordina trabajos temporizados y los clientes reciben preguntas, resultados y puntuaciones mediante eventos.

## 8. Pantallas y navegación

`AppNavigator` selecciona:

- `AuthStack`: `SignIn` y `Signup` cuando no hay sesión.
- `UserStack`: `UserDashboard` y `GameScreen` para usuarios autenticados no administradores.
- `AdminStack`: `AdminDashboard` para usuarios `ADMIN`.

Pantallas principales:

- `SignIn.screen.tsx`: correo, contraseña e inicio de sesión mediante `POST /auth/signIn`.
- `Signup.screen.tsx`: correo, nickname, contraseña y confirmación mediante `POST /auth/signUp`.
- `UserDashboard.screen.tsx`: entrada del usuario autenticado al flujo de jugador.
- `GameMainMenu.screen.tsx`: selección de modalidad/nivel y entrada a una sala.
- `GameLobby.screen.tsx`: espera de jugadores.
- `GamePlay.screen.tsx`: preguntas y respuestas en tiempo real.
- `GameResults.screen.tsx`: resultados de la partida.
- `AdminDashboard.screen.tsx`: entrada a funciones administrativas.
- `ManageCategories`, `CreateCategory` y `CategoryDetail`: gestión de categorías.
- `ManageQuestions`, `CreateQuestion` y `QuestionDetail`: gestión de preguntas y opciones.

Al añadir una pantalla, registrar la ruta en el stack correspondiente, documentar su rol permitido y enlazarla con el servicio o hook que consume. Ocultar un botón no sustituye la autorización del backend.

## 9. Reglas de negocio

- Los roles válidos son `PLAYER` y `ADMIN`.
- Un usuario nuevo pertenece inicialmente a `PLAYER`.
- El correo no debe duplicarse en la representación local.
- Una categoría requiere descripción, nivel y tipo; el formulario exige mínimo 10 caracteres en la descripción.
- Una pregunta debe tener texto o imagen, al menos dos opciones no vacías, una respuesta correcta y una categoría.
- El creador de una partida es su propietario y el único que puede iniciarla.
- Una partida solo se inicia en estado `WAITING`.
- No se puede unir a una partida ya iniciada o en preparación.
- Un jugador no se añade dos veces a la misma sala.
- Una desconexión marca al jugador como desconectado.
- Una respuesta inválida no actualiza la puntuación.
- Una respuesta correcta suma 100 puntos de partida; una incorrecta suma 0.
- Una revancha solo es válida cuando la partida está finalizada.
- La validación del cliente no reemplaza la validación del backend.

Al modificar roles, puntuación, estados, tiempos o acceso, actualizar API, cliente, eventos y pruebas del flujo completo.

## 10. Problemas frecuentes

### La API no inicia

Comprobar que existe `.env`, que Docker está activo, que PostgreSQL y Redis están levantados y que el puerto `3000` está disponible. Revisar variables obligatorias y logs de los contenedores.

### Error de PostgreSQL o migraciones

La API usa `synchronize: false` y `migrationsRun: false`. Ejecutar `yarn migration:show` y `yarn migration:run`. No borrar tablas ni modificar el historial para ocultar un fallo.

### El cliente no llega a la API

Desde un teléfono físico, `localhost` apunta al propio teléfono. Usar la IP del ordenador en `apiConfig.ts`, conectar ambos dispositivos a la misma red y comprobar firewall y puerto `3000`.

### Fallo de autenticación

Revisar variables de Cognito, verificación del usuario, credenciales, `accessToken` y la estrategia JWT. Para el juego, confirmar que el token se envía en `handshake.auth.token`.

### Socket.IO no conecta o la partida no avanza

Confirmar API y Redis, namespace `/game`, token válido, soporte WebSocket del proxy, existencia de la sala, propietario correcto, estado `WAITING`, trabajos BullMQ y listeners del cliente registrados una sola vez.

### Sesión inconsistente

El store restaura la sesión a partir de `accessToken`. Si el usuario no se carga, revisar `restoreSession`, `/auth/me` y el estado `user` antes de cambiar la navegación.

### El submódulo aparece modificado

Ejecutar `git -C <submódulo> status` para distinguir cambios internos de una referencia distinta al commit registrado. No sobrescribir trabajo ajeno.

## 11. Decisiones verificables

### Submódulos Git

Cliente, API y core se mantienen como repositorios independientes. Consecuencia: los cambios que cruzan componentes requieren actualizar más de un repositorio.

### Expo y React Native

Se eligieron para soportar Android, iOS y web. Los cambios móviles deben validarse en las plataformas afectadas.

### Migraciones TypeORM

PostgreSQL usa TypeORM con sincronización automática desactivada. Los cambios de esquema requieren migraciones revisadas.

### Redis para partidas

El estado temporal de las partidas se conserva en Redis. La entidad `Match` debe poder serializar y reconstruir sus snapshots.

### Socket.IO y BullMQ

Socket.IO gestiona la comunicación bidireccional y BullMQ los trabajos temporizados. Una modificación del flujo de juego puede afectar gateway, servicios, processor, eventos y cliente.

### AWS Cognito

La identidad y los grupos de usuario se delegan a Cognito. El entorno depende de una configuración válida de región, User Pool y Client ID.

No se encontraron ADR, política formal de ramas, plantilla obligatoria de Pull Request ni estándar de cobertura de pruebas. Esos puntos deben acordarse explícitamente antes de tratarlos como reglas del proyecto.

## 12. Criterio de finalización

Una tarea está lista cuando compila, las pruebas relevantes pasan, las migraciones están incluidas si aplican, los contratos se reflejan en sus consumidores, no se exponen secretos y la documentación queda alineada con el código.
