Vagrant setup for Docker
========================

## Bring up UCP and DTR nodes

```
vagrant up ucp-node dtr-node
```

## Join dtr-node to UCP cluster

```
docker run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join \
--admin-username admin \
--interactive \
--url https://172.28.128.5 \
--fingerprint 78:8E:80:46:33:F2:7A:20:1C:EE:08:C6:B2:CE:09:EE:1A:A7:92:A8:E2:9F:89:73:A8:5E:54:84:C2:D2:43:84 \
--host-address 172.28.128.6
```

## Install DTR

```
docker run -it --rm \
docker/dtr install \
--ucp-url https://172.28.128.5 \
--ucp-node dtr-node \
--dtr-external-url 172.28.128.6 \
--ucp-username admin --ucp-password admin \
--ucp-ca "$(cat ucp-ca.pem)"
```

## Connect to DTR from Mac or Linux client

https://docs.docker.com/docker-trusted-registry/configure/config-security/#install-registry-certificates-on-client-docker-daemons
