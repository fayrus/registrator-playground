# Registrator Playground

Local test environments for running [registrator](https://github.com/fayrus/registrator) against different backends.
Each subfolder contains a `docker-compose.yml` for a specific backend.

## Workflow

```bash
cd <environment>
docker compose up

# verify (see each environment below)

docker compose down
```

## Environments

### Consul

**Backend:** Consul 1.22.6 — `consul://`

```bash
curl http://localhost:8500/v1/agent/services | jq
```

```json
{
  "whoami-...": {
    "Service": "whoami",
    "Tags": ["test"],
    "Address": "172.x.x.x",
    "Port": 80
  }
}
```

---

### etcd

**Backend:** etcd 3.6 — `etcd://` (gRPC v3)

```bash
docker exec registrator-etcd-etcd-1 etcdctl get /services --prefix
```

```
/services/whoami/86a1706c0084:registrator-etcd-test-svc-1:80
192.168.97.2:80
```

---

### etcd-legacy

**Backend:** etcd 3.5 — `etcd-legacy://` (HTTP v2)

> Keys are written via the etcd v2 API and are not visible through `etcdctl`. Use the HTTP endpoint instead.

```bash
curl "http://localhost:2379/v2/keys/services?recursive=true" | jq
```

```json
{
  "action": "get",
  "node": {
    "key": "/services",
    "dir": true,
    "nodes": [
      {
        "key": "/services/whoami",
        "dir": true,
        "nodes": [
          {
            "key": "/services/whoami/<id>:registrator-etcd-legacy-test-svc-1:80",
            "value": "192.168.x.x:80"
          }
        ]
      }
    ]
  }
}
```

---

### ZooKeeper

**Backend:** ZooKeeper 3.9.3 — `zookeeper://`

```bash
docker exec registrator-zookeeper-zookeeper-1 zkCli.sh ls /services
```

```
[whoami]
```

---

### CoreDNS

**Backend:** CoreDNS 1.14.3 + etcd — `coredns://`

```bash
# Check entries in etcd
docker exec registrator-coredns-etcd-1 etcdctl get /skydns --prefix

# Resolve via DNS
dig @localhost -p 5300 whoami.service.local
```

```
;; ANSWER SECTION:
whoami.service.local.  300  IN  A  192.168.x.x
```

---

## Notes

- Versions are managed centrally in the root `.env` file.
- Backend infrastructure containers are excluded from service registration via `SERVICE_IGNORE=true`.
- Running multiple environments simultaneously is supported — each uses an isolated Docker network (`registrator-{backend}`).
