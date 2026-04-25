# 🐋 Docker Commands

### 🚀 Container Lifecycle
| Command | What it does |
| --- | --- |
| `docker run -d --name <name> <image>` | Run a container in the background (detached mode) without port mapping. |
| `docker run -d -p <host>:<container> --name <name> <image>` | Run a container and map a host port to a container port (e.g., `-p 8080:80`). |
| `docker ps` | List all currently running containers and their active port mappings. |
| `docker rm -f <name>` | Forcefully stop and remove a container. |

### 🔍 Inspection & Debugging
| Command | What it does |
| --- | --- |
| `docker inspect <name>` | Output all low-level configuration details of a container (in JSON format). |
| `docker inspect <name> \| grep IPAddress` | Quickly extract the internal IP address assigned to the container by Docker's IPAM. |
| `docker exec -it <name> <command>` | Execute a command *inside* the isolated container namespace (e.g., `docker exec -it web ip a`). |
