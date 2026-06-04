# Registrator Playground

Local test environments for running [registrator](https://github.com/fayrus/registrator) against different backends.
Each subfolder contains a `docker-compose.yml` for a specific backend.

## Workflow

1. Start the environment you want to test:
   ```
   cd consul
   docker compose up
   ```

2. Verify registered services (see table below for backend-specific commands).

3. Stop the environment:
   ```
   docker compose down
   ```

## Available environments

| Folder       | Backend             | Verify registrations                                                                    |
|--------------|---------------------|-----------------------------------------------------------------------------------------|
| `consul/`    | Consul 1.22.6       | `curl http://localhost:8500/v1/agent/services \| jq`                                   |
| `zookeeper/` | ZooKeeper 3.9.3     | `docker exec registrator-zookeeper-zookeeper-1 zkCli.sh ls /services`                  |
| `coredns/`   | CoreDNS 1.14.3      | `docker exec registrator-coredns-etcd-1 etcdctl get /skydns --prefix`                  |

## Notes

- All environments use `fayrus/registrator` from Docker Hub. Versions are managed in the root `.env` file.
- Backend infrastructure containers are excluded from service registration via `SERVICE_IGNORE=true`.
- Running multiple environments simultaneously is supported — each uses an isolated Docker network (`registrator-{backend}`).
