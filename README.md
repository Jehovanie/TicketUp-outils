# MinIO — stockage S3 des fichiers utilisateurs

## Demarrage

```bash
cp .env.example .env   # puis changer les mots de passe
docker compose up -d
docker compose logs minio-init   # verifier l'initialisation du bucket
```

- API S3 : http://localhost:9000
- Console web : http://localhost:9001 (login = `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`)

## Ce que fait `minio-init`

- cree le bucket `user-files` (prive, aucun acces anonyme)
- active le versioning, avec purge des anciennes versions apres 30 jours
- cree un utilisateur applicatif dont la policy est limitee a ce seul bucket
  (lecture / ecriture / suppression d'objets, pas d'admin)

L'application doit utiliser `APP_ACCESS_KEY` / `APP_SECRET_KEY`, jamais le compte root.

## Configuration cote application

```
S3_ENDPOINT=http://localhost:9000   # http://minio:9000 depuis un autre conteneur
S3_REGION=us-east-1
S3_BUCKET=user-files
S3_ACCESS_KEY=...
S3_SECRET_KEY=...
S3_FORCE_PATH_STYLE=true            # obligatoire avec MinIO
```

`S3_FORCE_PATH_STYLE` est indispensable : MinIO n'utilise pas les URLs
virtual-hosted (`bucket.host`) mais `host/bucket`.

Pour servir les fichiers aux utilisateurs, generer des URLs presignees
(`getSignedUrl` / `presigned_get_object`) plutot que de rendre le bucket public.

## Connecter une application du meme compose

Si l'app tourne dans un autre fichier compose, rattacher son service au reseau :

```yaml
networks:
  minio_net:
    external: true
    name: outils_minio_net
```

## Production

- mots de passe generes aleatoirement (`openssl rand -base64 32`)
- placer MinIO derriere un reverse proxy TLS et mettre `MINIO_SERVER_URL` en `https://`
- ne pas exposer le port 9001 (console) publiquement
- le volume `minio_data` contient les donnees : le sauvegarder
  (`mc mirror` vers un stockage externe, ou snapshot du volume)
