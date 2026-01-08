# Echidnaml Releases

Repositorio para la distribución de los instalables del software [EchidnaML](https://echidna.es/a-programar/echidnaml/).

En el apartado [Releases](https://github.com/EchidnaEducacion/echidnaml-releases/releases) se encuentran los instalables del software EchidnaML para los sistemas operativos:

- Linux (debian)
- Windows
- MacOS

EchidnaML es una plataforma integral para la programación con Scratch de placas de robótica educativa [Echidna](https://echidna.es) que incorpora el editor de modelos de [LearningML](https://learningml.org).

## Workflow: Compilación de instaladores (Build Installers from Private Repo)

Este repositorio incluye un workflow de GitHub Actions (.github/workflows/build-installers.yml) que automatiza la compilación y publicación de instaladores de EchidnaML. Resumen de su funcionamiento:

- Qué hace
  - Clona el repositorio privado EchidnaEducacion/echidnaml.
  - Compila instaladores para:
    - Linux (.deb)
    - macOS (.dmg)
    - Windows (.msi)
  - Sube los artefactos construidos como artifacts del workflow.
  - Si se dispone de una etiqueta/branch (tag), crea o actualiza una release en este repositorio con los instaladores.
  - Limpia artefactos antiguos del repositorio (borra los artifacts con más de 7 días).

- Triggers (cómo se dispara)
  1. Manualmente (desde la interfaz de GitHub)
     - Ve a la pestaña "Actions" → selecciona "Build Installers from Private Repo" → "Run workflow".
     - Input: `tag` (obligatorio). Por defecto: `main`.
     - El `tag` puede ser un nombre de rama o una etiqueta (por ejemplo `main` o `v1.2.3`). Si se especifica, además de compilar se intentará crear/actualizar la release con ese tag.

  2. Desde el repositorio privado (mediante repository_dispatch)
     - El workflow escucha eventos `repository_dispatch` de tipo `build-release`. El repositorio privado EchidnaEducacion/echidnaml puede dispararlo enviando un payload con `tag` en `client_payload`.
     - Ejemplo (desde el repo privado o CI con un token adecuado):
       ```bash
       curl -X POST \
         -H "Accept: application/vnd.github+json" \
         -H "Authorization: Bearer <PERSONAL_ACCESS_TOKEN>" \
         https://api.github.com/repos/EchidnaEducacion/echidnaml-releases/dispatches \
         -d '{"event_type":"build-release", "client_payload": {"tag":"v1.2.3"}}'
       ```
       - Sustituye `<PERSONAL_ACCESS_TOKEN>` por un token con permisos `repo` (o el scope mínimo necesario para dispatch).
       - El campo `client_payload.tag` es el valor que el workflow usará para checkout y para crear la release (si se proporciona).

- Tokens y permisos importantes
  - secrets.PRIVATE_REPO_TOKEN: token usado por actions/checkout para clonar el repositorio privado EchidnaEducacion/echidnaml. Debe tener acceso de lectura al repo privado.
  - secrets.GITHUB_TOKEN: usado por la acción que crea/actualiza la release y por la limpieza de artifacts. El workflow define permisos mínimos necesarios en los jobs.
  - Recomendación: limitar los scopes de los tokens al mínimo imprescindible y guardarlos en los Secrets del repositorio.

- Notas operativas
  - El job de build utiliza Node.js y ejecuta `yarn dist`; por tanto la compilación depende de la configuración del proyecto EchidnaML (package.json, electron-builder, etc.).
  - Los artifacts se mantienen por 7 días (retention-days: 7).
  - El job de cleanup borra artifacts del repositorio echidnaml-releases con más de 7 días utilizando la CLI `gh`.

---

*Actualizado por juanda mediante GitHub Actions assistant.*
