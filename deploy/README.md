# Deployment reference

The Compose example starts a clean Paper server and MariaDB using the image versions documented on ProDesk. It uses its own Compose project, Docker volumes, and a loopback port. The Argo CD manifests define the NGINX and monitoring applications for a K3s cluster.

## Docker Compose

From the repository root, create the local environment file. Choose three passwords in `.env` and set `MINECRAFT_EULA=TRUE` after accepting the Minecraft EULA.

```bash
cd deploy
cp -n .env.example .env
chmod 600 .env
```

Validate the Compose configuration and start the stack.

```bash
docker compose config --quiet
docker compose up --wait --wait-timeout 300
docker compose ps
docker compose exec minecraft rcon-cli list
```

The game server listens on `127.0.0.1:25566` on the Docker host. RCON is the server's administrative command protocol. The last command uses it from inside the Minecraft container. Docker provides a private network where Minecraft plugins can reach MariaDB at `mariadb:3306`.

For a new database volume, MariaDB creates `piraden_luckperms` and the `minecraft` database user. The bundled initialization SQL adds `piraden_litebans` and grants the same user access to that schema. Configure LuckPerms and LiteBans individually with their database name, the `minecraft` user, and the local `MARIADB_PASSWORD` if you add those plugins.

The live server stores Minecraft data under `/srv/data/minecraft-server`. This example stores its clean Paper world and MariaDB data in Docker-managed volumes. The [backup and restore notes](../README.md#automated-backups) show how the live data is archived and tested.

## K3s and Argo CD

Use a K3s cluster with Argo CD installed in `argocd` and the `local-path` StorageClass available. The Helm values expect a Secret named `monitoring-grafana-admin` with `admin-user` and `admin-password` keys. Store the values in two local files outside the checkout, each containing the value with no trailing newline.

On a fresh cluster, create the namespace and Secret. Replace the two file paths with your local paths.

```bash
sudo k3s kubectl create namespace monitoring
sudo k3s kubectl -n monitoring create secret generic monitoring-grafana-admin \
  --from-file=admin-user=/path/to/grafana-admin-user \
  --from-file=admin-password=/path/to/grafana-admin-password
```

From the repository root, apply the Application definitions and inspect the resulting resources.

```bash
sudo k3s kubectl apply -f deploy/argocd-applications.yaml
sudo k3s kubectl -n argocd wait --for=jsonpath='{.status.health.status}'=Healthy \
  application/hello-nginx application/monitoring --timeout=5m
sudo k3s kubectl -n argocd get applications
sudo k3s kubectl -n k3s-lab get deploy,pods,svc
sudo k3s kubectl -n monitoring get pods,pvc
```

`hello-nginx` reads the Kubernetes manifests under [apps/hello-nginx](../apps/hello-nginx/). `monitoring` uses the pinned `kube-prometheus-stack` Helm chart and the [values file](../apps/monitoring/values.yaml) from Git. Argo CD syncs both applications and reapplies the desired state if resources drift. When using a fork, point both Git sources in [argocd-applications.yaml](argocd-applications.yaml) at that fork and select its deployment branch.
