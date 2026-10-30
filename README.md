# docker-dnsmasq

dnsmasq in a docker container, configurable via a [simple web UI](https://github.com/jpillora/webproc)

[![Docker Pulls](https://img.shields.io/docker/pulls/jpillora/dnsmasq.svg)][dockerhub]
[![Image Size](https://images.microbadger.com/badges/image/jpillora/dnsmasq.svg)][dockerhub]

### Build with GitHub Actions (Ubuntu docker.io)

If your Ubuntu host cannot reach Docker Hub during a local build, use
[the GitHub Actions workflow](.github/workflows/docker-image.yml) to build on a
GitHub-hosted runner and publish to GitHub Container Registry (GHCR).
The Ubuntu host only needs access to GHCR to pull the finished image.
The image is currently **linux/amd64 only**, matching the webproc binary in
the Dockerfile.

1. Push this repository, including the workflow, to your own GitHub repository.
   Pushes to `main` or `master` trigger a build. You can also open **Actions >
   Build and publish Docker image > Run workflow** once the workflow is on
   the default branch. For forks, enable GitHub Actions if prompted.
2. The workflow uses the built-in `GITHUB_TOKEN` with `packages: write`;
   no Docker Hub account or manually configured publishing token is needed.
   It checks the dnsmasq configuration and webproc executable before publishing.
3. Images are published as `ghcr.io/<owner>/<repository>:sha-<full-commit-sha>`.
   Builds from the default branch also publish `:latest`. Names are lowercase;
   for this repository the image is `ghcr.io/nevermin/docker-dnsmasq:latest`.
4. For anonymous pulls, change the package visibility to **Public** in GitHub's
   package settings (new packages are private by default). For private packages,
   log in on Ubuntu using `sudo docker login ghcr.io -u YOUR_GITHUB_USERNAME`
   and supply a personal access token (classic) with `read:packages` when prompted.

On Ubuntu, install Docker if needed and pull the image:

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo docker pull ghcr.io/nevermin/docker-dnsmasq:latest
```

Replace the owner/repository if using a fork. Prepare `/opt/dnsmasq.conf` as
described below, replacing the example internal addresses with your own, then run:

```bash
# Replace this with the Ubuntu host's fixed LAN address.
DNS_IP=192.168.1.100
sudo docker run -d \
  --name dnsmasq \
  --restart unless-stopped \
  -p "$DNS_IP:53:53/udp" \
  -p "$DNS_IP:53:53/tcp" \
  -p 127.0.0.1:5380:8080 \
  -v /opt/dnsmasq.conf:/etc/dnsmasq.conf \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  -e HTTP_USER=admin \
  -e HTTP_PASS='REPLACE_WITH_A_STRONG_PASSWORD' \
  ghcr.io/nevermin/docker-dnsmasq:latest
```

Both UDP and TCP port 53 are needed for DNS. The web UI is available only on
the host at `http://127.0.0.1:5380`; use SSH forwarding for remote administration.
Do not forward DNS port 53 from the public Internet. Use the commit-specific
image tag instead of `latest` when you need a fixed deployment version.

### Usage

1. Create a [`/opt/dnsmasq.conf`](http://oss.segetech.com/intra/srv/dnsmasq.conf) file on the Docker host

   ```ini
   #dnsmasq config, for a complete example, see:
   #  http://oss.segetech.com/intra/srv/dnsmasq.conf
   #log all dns queries
   log-queries
   #dont use hosts nameservers
   no-resolv
   #use cloudflare as default nameservers, prefer 1^4
   server=1.0.0.1
   server=1.1.1.1
   strict-order
   #serve all .company queries using a specific nameserver
   server=/company/10.0.0.1
   #explicitly define host-ip mappings
   address=/myhost.company/10.0.0.2
   ```

1. Run the container

   ```
   $ docker run \
   	--name dnsmasq \
   	-d \
   	-p 53:53/udp \
   	-p 5380:8080 \
   	-v /opt/dnsmasq.conf:/etc/dnsmasq.conf \
   	--log-opt "max-size=100m" \
   	-e "HTTP_USER=foo" \
   	-e "HTTP_PASS=bar" \
   	--restart always \
   	jpillora/dnsmasq
   ```

1. Visit `http://<docker-host>:5380`, authenticate with `foo/bar` and you should see

   <img width="833" alt="screen shot 2017-10-15 at 1 41 21 am" src="https://user-images.githubusercontent.com/633843/31580966-baacba62-b1a9-11e7-8439-ca1ddfe828dd.png">

1. Test it out with

   ```
   $ host myhost.company <docker-host>
   Using domain server:
   Name: <docker-host>
   Address: <docker-host>#53
   Aliases:

   myhost.company has address 10.0.0.2
   ```

#### MIT License

Copyright &copy; 2018 Jaime Pillora &lt;dev@jpillora.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[dockerhub]: https://hub.docker.com/r/jpillora/dnsmasq/
