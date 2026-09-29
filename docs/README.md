# Documentación de LinguaPlay

## Documentos

- [Documentación técnica](technical-documentation.md): arquitectura, tecnologías, datos, API, instalación, pruebas, despliegue y mantenimiento.
- [Documentación interna](internal-documentation.md): incorporación, flujo de desarrollo, Git, reglas de negocio, flujos funcionales, interfaces y solución de problemas.

## Diagramas (`diagrams/`)

Los fuentes son PlantUML (`.puml`); `diagrams/png/` tiene la exportación de cada uno. Todos reflejan el código actual de LP-API y LP-MOB.

| Archivo | Tipo | Contenido |
|---|---|---|
| `use-case.puml` | Casos de uso | Jugador y Administrador: cuenta, trivia, historieta, catálogo, likes, administración y moderación. |
| `class-diagram.puml` | Clases | Entidades TypeORM con sus relaciones, enumeraciones y dominio de la trivia en Redis. |
| `relational-model.puml` | Modelo relacional | Tablas de PostgreSQL con claves, `ON DELETE`, índices y restricciones únicas (según las migraciones). |
| `entity-relationship-chen.puml` | Entidad-relación (Chen) | Entidades, relaciones y atributos principales. |
| `component-diagram.puml` | Componentes | Features de LP-MOB, módulos de LP-API y servicios externos. |
| `deployment-diagram.puml` | Despliegue | Contenedores de `deploy/docker-compose.prod.yml`, clientes y servicios de AWS y Cloudflare. |
| `structure-project.puml` | Estructura | Carpetas del repositorio y de cada aplicación. |
| `sequence-auth.puml` | Secuencia | Registro, inicio de sesión y renovación con Cognito. |
| `sequence-game-flow.puml` | Secuencia | Partida de trivia: sala, ciclo de preguntas con BullMQ y resultados. |
| `sequence-story-flow.puml` | Secuencia | Historieta: lobby, turnos con revisión de IA, generación de media, review y guardado. |
| `sequence-story-moderation.puml` | Secuencia | Moderación: quitar, restaurar y regenerar imágenes. |
| `sequence-catalog-engagement.puml` | Secuencia | Catálogo: lectura, reacciones por viñeta y likes. |

## Cómo exportarlos

Con la extensión "PlantUML" de VS Code: abrir un `.puml` y usar la vista previa.

Por línea de comandos (requiere Java; sin Graphviz se usa el layout Smetana):

```bash
java -jar plantuml.jar -charset UTF-8 -Playout=smetana -tpng -o png docs/diagrams/*.puml
```
