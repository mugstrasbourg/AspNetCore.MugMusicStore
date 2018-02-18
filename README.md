# [Lab] Déploiement d'une application ASP.NET Core avec Docker sur Azure

## Docker

Maintenant que le site est corrigé et est parfaitement fonctionnel sous IIS Express, voyons voir s'il n'est pas possible de le faire tourner sur Docker.

Prérequis :
- [Docker pour Windows](https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe)

Attention aux machines Windows 10 Insider build 17063+, il faudra installer cette version de Docker : https://download.docker.com/win/stable/14687/Docker%20for%20Windows%20Installer.exe
Issue sur GitHub : https://github.com/docker/for-win/issues/1458

1) Commençons ! Ajouter le support de Docker à votre projet.
Pour cela, faites un clic droit sur le projet Web, dans "Ajouter" cliquer sur "Docker Support".
Un nouveau projet **docker-compose.dcproj** est ajouté à votre solution et contient les fichiers suivants : 
- **.dockerignore**
- **docker-compose.yml**
- **docker-compose.override.yml**

Il y a également un fichier **Dockerfile** qui a été ajouté au projet web.

Lorsque vous lancez l'opération pour la première fois, cela peut durer quelques minutes le temps de récupérer tous les packages nécessaires au lancement de l'application.

2) Définir le projet **docker-compose** comme projet de démarrage puis lancer celui-ci.
Que se passe t-il ?

3) Pour gérer la connexion à SQL Server, le fichier **docker-compose.yml** doit être modifié.
L'image de SQL Server étant relativement conséquente à télécharger, on pourra directement passer au point 4)

**Windows**

      db:
        image: "microsoft/mssql-server-windows-express"
        environment:
          SA_PASSWORD: "MugStr@sbourg"
          ACCEPT_EULA: "Y"

**Linux**

      db:
        image: "microsoft/mssql-server-linux"
        environment:
          SA_PASSWORD: "MugStr@sbourg"
          ACCEPT_EULA: "Y"

L'application MusicStore ne pourra pas démarrer tant que la base de données n'aura pas été initialisée. 
Il est ainsi nécessaire de modifier la définition de musicstore en rajoutant **depends_on** :

      musicstore:
        image: musicstore
        build:
          context: .
          dockerfile: samples\MusicStore\Dockerfile
        depends_on: 
          - db
    
Modifier la chaîne de connexion à la base de données en conséquence dans le fichier **config.json**

    "ConnectionString": "Server=db;Database=MusicStore;User=sa;Password=MugStr@asbourg;Trusted_Connection=True;MultipleActiveResultSets=true;Connect Timeout=30;"

Relancer le projet. Cela peut prendre quelques minutes le temps de récupérer l'image Docker pour SQL Server.

4) Modifier le fichier **Startup.cs** et remplacer SQL Server par SQLite.

```csharp
services.AddDbContext<MusicStoreContext>(
    options => options.UseSqlite(Configuration["Data:DefaultConnection:SqliteConnectionString"]));
```
    
5) Dans le fichier **config.json**, en dessous de la chaîne de connexion pour SQL Server, ajouter une ligne pour la connexion à la base de données SQLite :

```json
"SqliteConnectionString": "Data Source=musicStore.db"
```

6) Relancer le projet sur Docker afin de vérifier que tout est fonctionnel.

1ère étape accomplie !
