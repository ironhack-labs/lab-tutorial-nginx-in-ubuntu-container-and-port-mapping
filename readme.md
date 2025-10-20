![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Lab | Nginx in Ubuntu container — first without, then with port mapping

Containers have their **own** network stack. A service listening on `localhost:80` *inside* a container isn’t reachable from the host unless you **map a host port** to a **container port**.
With Docker this is done using `-p HOST_PORT:CONTAINER_PORT` (e.g., `-p 8080:80`). Without it, `curl localhost` on the host will fail, even though the service is healthy inside the container.

## Prerequisites

* Docker installed and running.
* Internet access to install packages inside the container.

## Part 1 — Run Nginx **without** port mapping (host curl should fail)

1. **Start a plain Ubuntu container (no port mapping)**

```bash
docker run -dit --name web-nomap --rm ubuntu
```

2. **Install Nginx inside the container**

```bash
docker exec web-nomap apt-get update
docker exec web-nomap apt-get install -y nginx curl
```

3. **Change the default HTML**

```bash
docker exec web-nomap bash -lc 'echo "<h1>Hello from INSIDE the container (no mapping)</h1>" > /var/www/html/index.html'
```

4. **Start Nginx (foreground mode)**

```bash
docker exec -d web-nomap nginx -g "daemon off;"
```

> `-d` runs the process detached so your terminal doesn’t block.

5. **Verify from *inside* the container (should work)**

```bash
docker exec web-nomap curl -s http://localhost | head -n1
```

Expected: `"<h1>Hello from INSIDE the container (no mapping)</h1>"`

6. **Try from the *host* (should FAIL)**

```bash
curl -v http://localhost
```

Expected: `Connection refused` or similar.
**Why?** Nginx is listening on the container’s `localhost:80`, which is not bound to your host network. No port mapping = no reachability from the host.

7. **Clean up Part 1**

```bash
docker rm -f web-nomap
```

---

## Part 2 — Run Nginx **with** port mapping (host curl should work)

1. **Start a new Ubuntu container with port mapping**

```bash
docker run -dit --name web-mapped --rm -p 8080:80 ubuntu
```

> `-p 8080:80` = host port 8080 → container port 80.

2. **Install Nginx**

```bash
docker exec web-mapped apt-get update
docker exec web-mapped apt-get install -y nginx curl
```

3. **Change the HTML**

```bash
docker exec web-mapped bash -lc 'echo "<h1>Hello from a mapped port (8080→80)</h1>" > /var/www/html/index.html'
```

4. **Start Nginx (foreground)**

```bash
docker exec -d web-mapped nginx -g "daemon off;"
```

5. **Verify from *inside* the container**

```bash
docker exec web-mapped curl -s http://localhost | head -n1
```

Expected: `"<h1>Hello from a mapped port (8080→80)</h1>"`

6. **Verify from the *host* (should WORK now)**

```bash
curl -s http://localhost:8080 | head -n1
```

Expected: `"<h1>Hello from a mapped port (8080→80)</h1>"`

7. **(Optional) See the port binding**

```bash
docker ps --filter name=web-mapped
# or
docker inspect -f '{{json .NetworkSettings.Ports}}' web-mapped | jq
```

8. **Clean up**

```bash
docker rm -f web-mapped
```

## Submission Guidelines

To complete this lab:

1. Take **screenshots** of the Nginx container running and the port mapping.
    - `part1-fail.png` — screenshot showing curl localhost failing from the host (no port mapping).
    - `part2-success.png` — screenshot showing curl localhost:8080 succeeding from the host (with port mapping).
2. Paste the screenshots into a **Google Doc**.
3. Upload the document to **Google Drive**.
4. To submit the lab, paste the **Google Drive link** in the submission field in the Student Portal.

---

## Extra

### Quick discussion prompts

* Why did `curl localhost` fail in Part 1 but succeed in Part 2?
* What happens if you use `-p 80:80` instead and then visit `http://localhost`?
* How would multiple containers avoid clashing on host ports?

### Common pitfalls & fixes

* **Nginx “not found”** → forgot `apt-get update` before `apt-get install`.
* **`curl localhost` on host still fails** → confirm you started the mapped container (`docker ps`) and you are curling the right host port (`:8080`).
* **Nginx exits immediately** → always start it with `nginx -g "daemon off;"` so it stays in the foreground (containers stop when the main process exits).
