# LinguaPlay

LinguaPlay es una aplicación para practicar inglés jugando. Tiene dos modos:

- **Trivia:** preguntas por nivel del MCER (A1 a C2), sola o en salas multijugador en tiempo real.
- **Historieta:** los jugadores escriben juntos una historieta, viñeta por viñeta. La IA revisa el inglés de cada viñeta, narra el texto y dibuja las imágenes.

Las historietas terminadas se guardan. Todos pueden explorarlas, reaccionar a cada viñeta y darles like, y los administradores las moderan.

## Estructura del proyecto

El repositorio usa **Git Submodules**: cada aplicación es un repositorio independiente.

```
LinguaPlay
├── apps
│   ├── LP-API            # API NestJS (submódulo)
│   └── LP-MOB            # App Expo / React Native: Android, iOS y web (submódulo)
├── linguaplay-core       # tipos y contratos compartidos (submódulo; hoy no lo importan las apps)
├── deploy                # despliegue con Docker Compose
├── docs                  # documentación y diagramas UML
└── LICENSE
```

## Documentación

- [Manual técnico](docs/manual-tecnico.md), [manual de usuario](docs/manual-usuario.md) y [ficha técnica](docs/ficha-tecnica-dnda.md).
- [Documentación técnica](docs/technical-documentation.md) y [documentación interna](docs/internal-documentation.md) del equipo.
- [Diagramas UML](docs/README.md) en `docs/diagrams/`.
- README de cada aplicación: [LP-API](apps/LP-API/README.md) y [LP-MOB](apps/LP-MOB/README.md).

## Puesta en marcha (desarrollo)

Requisitos: Git, Node.js 22 (el mínimo es 20), Yarn 1 y Docker Desktop. Para el móvil, además, Android Studio o Expo Go.

```bash
git clone --recurse-submodules https://github.com/smyleface18/LinguaPlay.git
cd LinguaPlay

# API
cd apps/LP-API
yarn install
cp .env.template .env          # completar (PostgreSQL, Redis, AWS, IA)
docker compose up -d           # PostgreSQL y Redis
yarn migration:run
yarn start:dev                 # http://localhost:3000

# App (en otra terminal)
cd apps/LP-MOB
yarn install
cp .env.example .env           # EXPO_PUBLIC_API_BASE_URL=http://localhost:3000
yarn web                       # o yarn android
```

## Despliegue

Un servidor con Docker levanta la API, la versión web, PostgreSQL y Redis:

```bash
cp deploy/.env.prod.example deploy/.env.prod     # completar; no se versiona
docker compose -f deploy/docker-compose.prod.yml --env-file deploy/.env.prod up -d --build
```

- La API queda en el puerto `API_PORT` (3000) y corre las migraciones al arrancar.
- La web queda en `WEB_PORT` (8080).
- Para producción conviene poner delante un proxy con HTTPS.

El detalle está en el manual técnico.

## Trabajo con submódulos

```bash
git submodule update --init --recursive     # después de clonar sin --recurse-submodules
git pull && git submodule update --init --recursive
git submodule status                        # commit de cada submódulo
```

Flujo de un cambio:

1. Commit y push dentro del submódulo (por ejemplo, `apps/LP-API`).
2. En la raíz, registrar el commit nuevo:

   ```bash
   git add apps/LP-API
   git commit -m "chore(submodules): update LP-API"
   git push
   ```

Los submódulos no se actualizan solos. Cada uno tiene su propio historial.
