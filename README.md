# cyber-trivy

Run [Trivy](https://trivy.dev/) as a hosted scanner so multiple Trivy clients
can use a centrally managed vulnerability database when scanning container
images.

## How it works

The Trivy server manages the vulnerability database. A Trivy client connects to
the server to scan an image; the client still needs access to the image registry
(and credentials for private registries).

## Run the server

Install Docker on a host that can reach the internet to download Trivy's
vulnerability database. Choose a Trivy release and use that same version for
the server and clients:

```sh
export TRIVY_VERSION=0.XX.X # replace with the release you selected
docker run -d \
  --name trivy-server \
  --restart unless-stopped \
  -p 127.0.0.1:8080:8080 \
  aquasec/trivy:${TRIVY_VERSION} \
  server --listen 0.0.0.0:8080
```

The port is bound to localhost in this example. To make the service available
to other machines, put it behind a TLS-enabled reverse proxy or private
network, and configure the proxy/network to allow only trusted clients. Do not
expose the Trivy server directly to the public internet. Check startup and
database update logs with:

```sh
docker logs -f trivy-server
```

## Scan an image

Install the matching Trivy version on a client machine, then point an image
scan at the server:

```sh
trivy image --server https://trivy.example.com alpine:3.21
```

Replace the example hostname and image with your server address and target.
Use `http://` only on a trusted private network; use the HTTPS endpoint provided
by your reverse proxy for remote clients.

To fail a CI job when a scan finds fixable critical or high vulnerabilities:

```sh
trivy image \
  --server https://trivy.example.com \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  alpine:3.21
```

## Operations and security

- Pin the server and client to the same Trivy release, and update them together.
- Keep the server patched and monitor its logs and database update status.
- Restrict network access to trusted clients and terminate TLS at a trusted
  proxy or load balancer.
- Configure registry credentials on the client that accesses a private image;
  do not put credentials in image names or command examples committed to source.
- See the [Trivy client/server documentation](https://trivy.dev/latest/docs/references/modes/client-server/)
  and [Trivy installation guide](https://trivy.dev/latest/getting-started/installation/)
  for current options and supported installation methods.