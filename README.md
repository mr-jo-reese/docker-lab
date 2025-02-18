# Docker-lab example
Here is an example of how to create your own docker lab in your local network with your own CA. For this example you need a Linux server with Docker installed.

# Docker Services
* Traefik
* Dockge
* Portainer
* Nginx (test)
* IT-Tools
* Sitrling-PDF
* Vaultwarden
* NetBox

## Docker-directory
Create your own docker directory and clone this repository. See the example below.

```bash
mkir /opt/docker && cd $_

git clone https://github.com/mr-jo-reese/docker-lab.git
```
## Self-certificat CA
To create your own CA, you can use tools like mkcert or create a certificate with openssl.

**Links:**
* [mkcert](https://github.com/FiloSottile/mkcert)
* [openssl](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-ca-on-ubuntu-20-04)

### Create CA and certificate with
```bash
# Install mkcert on Ubuntu
apt install mkcert

# Create your CA
mkcert --install

# Create wilde-card certificat for Traefik
mkcer *.intern.local
```
The certificates are created in your user home folder.

## Trarfik and DNS-Record
When the certificate and private key are ready, move them to the traefik directory "traefik/certs".

To reach the docker with the browser, you can add the domain names to your internal DNS server or to "/etc/hosts". Do not forget the IP address of the server running docker in this example (192.168.1.99).

### Example /etc/hosts
```bash
192.168.1.99    traefik.intern.local
192.168.1.99    dockge.intern.local
192.168.1.99    portainer.intern.local
.
.
.
.
.
```

