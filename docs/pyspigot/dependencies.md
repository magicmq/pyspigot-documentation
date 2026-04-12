# Internal Dependency Management

Starting in version 0.10.0, PySpigot uses an internal runtime dependency management system to download and load the third-party libraries it depends on. Rather than bundling ("shading") these libraries directly into the plugin JAR at compile time, they are downloaded from a remote repository the first time the plugin loads and cached locally for all subsequent starts.

## Why Runtime Dependency Management?

Historically, Minecraft plugins bundled all of their dependencies directly into a single "fat" JAR file. While this works, it has several drawbacks that PySpigot's runtime dependency management system avoids:

- **Smaller JAR size.** Bundling large libraries into the JAR inflates its file size significantly. By downloading them separately, the PySpigot JAR file remains much smaller.
- **Fewer classpath conflicts.** When two plugins bundle the same library at different versions, Java can only load one version, which often causes subtle errors or incompatibilities. Downloading dependencies separately (and remapping their package names at runtime) greatly reduces this risk.
- **Automatic dependency updates.** When PySpigot is updated to require a newer version of a dependency, that new version is downloaded automatically on next startup. Old versions are cleaned up automatically as well. No manual intervention is required.
- **Platform flexibility.** On platforms that do not bundle certain JDBC drivers (such as Velocity and BungeeCord), PySpigot can download and provide them automatically, eliminating the need for server admins to manually source and install them.
- **Integrity verification.** Every downloaded file is verified against a SHA-256 checksum before being added to the classpath, ensuring the files are authentic and unmodified.

## Internet Connection Requirement

Because dependencies are downloaded from a remote repository, **an active internet connection is required the first time PySpigot loads** on a given server. Dependencies are downloaded from a self-hosted Maven Central mirror (`repo.magicmq.dev`) first, falling back to Maven Central directly if the mirror is unavailable.

Once all dependencies have been downloaded and cached locally, no internet connection is required for subsequent server starts — unless PySpigot is updated to a version that requires new or updated dependency versions, in which case those changes are downloaded automatically on the next startup.

## The `internal` Folder

Downloaded dependencies are stored in the `java-libs/internal` subfolder within PySpigot's plugin folder:

```
plugins/PySpigot/
└── java-libs/
    └── internal/
        ├── HikariCP-7.0.2.jar
        ├── HikariCP-7.0.2-relocated.jar
        ├── mongodb-driver-sync-5.6.0.jar
        ├── mongodb-driver-sync-5.6.0-relocated.jar
        └── ...
```

Each dependency is stored as two files: the original downloaded JAR and a remapped copy (suffixed with `-relocated`). The remapped copy has the library's internal package names rewritten to avoid classpath conflicts with other plugins that may use the same library.

???+ warning

    **Do not modify, rename, or delete any files in the `java-libs/internal` folder.** These files are managed entirely by PySpigot. Tampering with them may prevent the plugin from loading or cause unexpected runtime errors. If the folder or its contents are accidentally corrupted, simply delete the `internal` folder and restart the server — PySpigot will re-download everything automatically.

## Managed Dependencies

The following is a high-level summary of the libraries PySpigot manages through this system:

| Library | Purpose |
|---|---|
| HikariCP | SQL database connection pooling (MySQL, MariaDB, etc.) |
| MongoDB Java Driver | MongoDB database connectivity |
| Lettuce | Redis client |
| Netty | Networking library (required by Lettuce) |
| Reactor Core | Reactive programming library (required by Lettuce) |
| bStats | Plugin metrics (Bukkit platform) |
| Adventure API | Text/component library (Bukkit platform) |
| MySQL Connector/J | MySQL JDBC driver (Velocity and BungeeCord only) |
| SQLite JDBC | SQLite JDBC driver (Velocity and BungeeCord only) |

???+ note

    On Bukkit-based servers (Spigot, Paper, etc.), MySQL and SQLite JDBC drivers are already bundled with the server software and are not downloaded by PySpigot. On Velocity and BungeeCord, these drivers are not bundled, so PySpigot downloads them automatically.
