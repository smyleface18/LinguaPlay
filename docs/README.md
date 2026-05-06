# LinguaPlay Diagrams (PlantUML + Mermaid)

This folder contains the core diagrams used in the presentation. They focus on LP-API + LP-MOB and cover all major modules.

## Diagram list

- use-case.puml: Use case diagram (actors and functional requirements).
- class-diagram.puml: Class diagram (entities + key domain classes).
- sequence-auth.puml: Sequence diagram for sign-in with AWS Cognito.
- sequence-game-flow.puml: Sequence diagram for real-time game flow (create/join/start/questions).
- component-diagram.puml: Component diagram (mobile, API, external services, data stores).
- deployment-diagram.puml: Deployment diagram (runtime nodes and connections).
- relational-model.puml: Relational model (ER diagram in PlantUML).

## How to render

Option A: VS Code extension

1. Install "PlantUML" extension.
2. Open any .puml file.
3. Use the preview command from the editor.

Option B: CLI (PlantUML JAR)

1. Download plantuml.jar.
2. Run: java -jar plantuml.jar docs/diagrams/\*.puml

Option C: PlantUML (ER diagram)

1. Open docs/diagrams/relational-model.puml.
2. Use the PlantUML preview command.

Notes:

- The diagrams are based on the current codebase (LP-API + LP-MOB).
- If you rename modules or add new ones, update the diagrams accordingly.
