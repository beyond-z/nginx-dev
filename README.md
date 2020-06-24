# nginx-dev

## Overview
This repo is a nginx reverse proxy for the development environment. It allows
you to hit the local dev URLs without needing to put in the port. It's also a handy
reference for the port numbers for each dev application.

## Troubleshooting
Occasionally you'll get a `Bad Gateway nginx 1.17.4` error. Not sure why, but if you just restart the
service using `./docker-compose/scripts/restart.sh` it should work.

## SSL Support
Sometimes it's useful to be able to use HTTPS/SSL in the local development environment.
For example, if you're writing an LTI extension which only supports SSL or if you're troubleshooting
an SSL related bug.

Instead of adding support in each application and switching to boot it in SSL mode, the nginx reverse
proxy allows an easy way to run the app in HTTP mode but to be able to hit it in SSL mode. Here is how you
can setup SSL support for a given app, in this example for the Braven Platform which runs locally
at http://platformweb:3020

1. First, create a self-signed SSL cert using the following command (substitue platformweb for your
app name):
```
openssl req \
    -newkey rsa:2048 \
    -x509 \
    -nodes \
    -keyout platformweb-selfsigned.key \
    -new \
    -out platformweb-selfsigned.crt \
    -subj /CN=platformweb \
    -reqexts SAN \
    -extensions SAN \
    -config <(cat /System/Library/OpenSSL/openssl.cnf \
        <(printf '[SAN]\nsubjectAltName=DNS:platformweb')) \
    -sha256 \
    -days 3650
```

2. Check the .crt and .key into src control.

3. Update docker-compose.yml to map your new .crt and .key into the docker image as a volume. See the platformweb example in there.

4. Then on a Mac open up Keychain Access app, click on Certificates on the left, drag your new `platformweb-selfsigned.crt` into there, double click it, expand the Trust section, change the setting to *"When using this certificate" : "Always Trust"*, close and type your computer password to save. 

5. Update the `nginx.conf` to liston on port 443 and use your new self-signed key which you've told your local computer to trust. Just look at the platformweb config in there for an example

