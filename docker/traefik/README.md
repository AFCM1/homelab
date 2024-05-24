# What's Traefik?

Traefik is a modern open-source reverse proxy and ingress controller that simplifies the deployment, routing, and load balancing of microservices.
Some others popular alternatives for homelabs are :  
- [Caddy](https://github.com/caddyserver/caddy) - Extremely easy to setup
- [NGINX Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager) - GUI
- [Swag](https://github.com/linuxserver/docker-swag) - Have fail2ban configured with it

## Notes
- [Porkbun](https://porkbun.com/) is my domain name registrar/provider.  
- A list of all supported providers can be found [here](https://doc.traefik.io/traefik/https/acme/#providers).  
- I'm using dnsChallenge for validating domain ownership when obtaining SSL certificates, especially when wildcard certificates are needed.

## Short tutorial

1️⃣ Create the [docker-compose.yml file.](https://github.com/AFCM1/my_homelab/blob/main/docker/traefik/docker-compose.yml)  

2️⃣ Retrive your Porkbun secret and API key [here.](https://porkbun.com/account/api)  

3️⃣ You will need to add them in the .env file.

> PORKBUN_API_KEY: ${PORKBUN_API_KEY}  
> PORKBUN_SECRET_API_KEY: ${PORKBUN_SECRET_API_KEY}

4️⃣ Use the docker command to create a custom network, I'm going to call it *proxy*.
> docker network create proxy

5️⃣ Create a file called *acme.json* in folder containing your docker-compose file. The acme.json will contain the SSL certificate information.  
> touch /traefik/acme.json && chmod 600 /traefik/acme.json

*Perms 600 ensures that only the file owner can read and modify them. Other users, including group members, won’t have access to these files. 600 is usually set for confidential files.*  

6️⃣ We will use Docker Socket Proxy. Uncomment the second part of the docker-compose file. I already had a running docker socket proxy because I'm using it for [homepage](https://github.com/gethomepage/homepage)  

Why a docker socket proxy ?  

*Imagine you have a publicly accessible container (like a box) that runs Docker. This container can talk to Docker’s API (like a special phone line to control Docker).
Now, here’s the catch: Giving this container full access to the Docker API is risky. It’s like giving someone a powerful tool without any restrictions.  
To make things safer, we introduce a proxy container (like a middleman). This proxy controls what the first container can say to the Docker API. It only allows through what’s necessary for things to work. So, we limit the container’s access 🔒
By doing this, we balance security and functionality. It’s like having a guard at the door who checks IDs before letting anyone in.*  

7️⃣ Start up the container  
> docker-compose up -d

8️⃣ To redirect your services, edit your containers docker compose files and add a new section called "labels:" and these keys-values :  
> labels:  
      - "traefik.enable=true" #When set to true, Traefik will automatically configure routing rules and handle incoming requests for this service   
      - "traefik.docker.network=proxy" #the container must be in the same network than traefik  
      - traefik.http.middlewares.redirect-https.redirectScheme.scheme=https #It ensures that incoming HTTP requests are redirected to HTTPS (secure) connections    
      - traefik.http.middlewares.redirect-https.redirectScheme.permanent=true #Indicates that the redirection should be permanent  
      - "traefik.http.routers.app.rule=Host(`{SUBDOMAIN_NAME}`)" #app.exemple.com  
      - "traefik.http.routers.app.entrypoints=https" #Specifies that the router should listen on the https entrypoint (port 443)  
      - "traefik.http.routers.app.tls=true" #Enables TLS (SSL) termination for this router, allowing secure communication over HTTPS  
      - "traefik.http.services.app.loadbalancer.server.port={CONTAINER_PORT}" #Specifies that the app service listens on port xyz

⚠️ WARNING

You must replace "app" with the name of your application.  

9️⃣ Restart your container and access your app with its subdomain name.

For more details: [doc.traefik.io](https://doc.traefik.io/traefik/)
