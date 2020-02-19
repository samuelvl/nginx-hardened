# Nginx hardened

Deploy nginx with optimal configuration and enforced security.

## Basic authentication

To access to `https://www.example.com/settings` authentication is required.

1. Generate a password for admin user.

```
htpasswd -c -b -5 .htpasswd admin <password>
```

## Enable TLS

1. Generate a CSR (Certificate Signing Request).

```
openssl req -nodes -new -sha512 -newkey rsa:4096 -days 365 \
    -keyout certs/nginx.key -out certs/nginx.csr
```

2. Auto-approve the CSR.

```
openssl x509 -req -in certs/nginx.csr -signkey certs/nginx.key \
    -out certs/nginx.crt
```

3. Generate custom DH parameters.

```
openssl genpkey -genparam -algorithm DH -pkeyopt dh_paramgen_prime_len:4096 \
    -out certs/dhparam.pem
```

## OCI image

1. Build OCI image.

```
podman build -t nginx:1.14.1 -f oci/Dockerfile .
```

2. Run nginx container.

```
podman run --name nginx --detach --rm --tty --publish 8080:8080 --publish 8443:8443 \
    --volume "$(pwd)/certs/nginx.crt:/etc/pki/tls/certs/nginx.crt:z" \
    --volume "$(pwd)/certs/nginx.key:/etc/pki/tls/private/nginx.key:z" \
    --volume "$(pwd)/certs/dhparam.pem:/etc/pki/tls/certs/dhparam.pem:z" \
    --volume "$(pwd)/.htpasswd:/etc/nginx/.htpasswd:z" nginx:1.14.1
```

## References

- https://medium.com/@callback.insanity/forwarding-nginx-logs-to-docker-3bb6283a207
- https://github.com/denji/nginx-tuning
- https://www.linode.com/docs/web-servers/nginx/tls-deployment-best-practices-for-nginx/
- https://tools.ietf.org/html/rfc7540
- https://tools.ietf.org/html/rfc5114