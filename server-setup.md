## Configuring Core Server Components
<i class="fa-regular fa-square-check"></i> Networking <br>
<i class="fa-regular fa-square-check"></i> VPN <br>
<i class="fa-regular fa-square-check"></i> Docker <br>
<i class="fa-regular fa-square-check"></i> Docker Ecosystem <br>


### Docker Core System
With docker installed on the system we can now setup the admin and proxy networks, and our critical systems

<details>
  <summary>Networking Setup</summary>
  <div id="expand" style="border: 1px">
    
  ```
  docker network create -subnet 172.22.22.0/24 admin 
  docker network create -subnet 172.22.21.0/24 proxy
  ```
  
<i class="fa-regular fa-square-check"></i> Open [Tailscale Management Console](https://login.tailscale.com/admin/machines) and under the server config, setup Subnets <br>

  <i class="fa-regular fa-square-check"></i> Run a test container on each docker network<br>
    <i class="fa-regular fa-square-check"></i> make sure you can access both containers from your home location<br>
    <i class="fa-regular fa-square-check"></i> make sure you can ping the containers on just their hostnames<br>
    <i class="fa-regular fa-square-check"></i> ensure that you can ping from the containers out to the internet<br>
  
    After docker is installed, we will create our external networks and assign ranges
  </div>
  
 </details>
<hr>  
   <details>
      <summary>Verify Network Connectivity</summary>
      
  ```
      git clone https://github.com/nieleyde/tutum-hello-world.git && cd tutum-hello-world
      sed -i 's/php-fpm/php-fpm83/g' Dockerfile
      docker build --network host -t hello-world .
      
      sudo docker run --hostname admin -d --network=admin -p 80 tutum/hello-world
      sudo docker run --hostname proxy -d --network=proxy -p 80 tutum/hello-world
        
      docker exec -it admin ping ibm.com
      docker exec -it admin ping 100.100.100.100      
      docker exec -it admin ping 172.22.21.1
      **all the above should respond wirh nirmal ping**
        PING 172.22.22.1 (172.22.22.1): 56 data bytes
        64 bytes from 172.22.22.1: seq=0 ttl=64 time=0.198 ms
        64 bytes from 172.22.22.1: seq=1 ttl=64 time=0.145 ms

      ** the final test should fail - as there is no direct communication between containers on different networks **
      docker exec -it admin ping proxy    
        PING 172.22.22.4 (172.22.22.4): 56 data bytes
      
      docker exec -it proxy ping ibm.com
      docker exec -it proxy ping 100.100.100.100
      docker exec -it admin ping 172.22.22.1     
      docker exec -it admin ping admin
  ```    
</details>

<hr>  

<details>
  <summary>Define standard templates etc</summary>
  To esnure that the system doesnt become unmanagable, setup templates
  
   <i class="fa-regular fa-square-check"></i> all docker folders and processes will be owned by docker (666) and group docker (666)<br>
   <i class="fa-regular fa-square-check"></i> your default user should be added to docker group <br>
  
  | filesystem folder | purpose |
  | -- | -- |
  |  ```/etc/user/config``` | any volumes mappingfor containers will persist here |
  |  ```/etc/user/archive``` | |
  |  ```/etc/user/github``` | any folder that is to be sync'd with github should store its .git cache here |
  |  ```/etc/user/docker``` | any manual docker setups (such as core) should be here |
  |  ```/etc/user/dev``` | aa |
  
  
    eg - creating a new container that may have other files and be built locally
    mkdir -p /etc/pknw1/docker/container
    git init --separate-git-dir /etc/pknw1/github/container.git
    
    the persistent files or folders should always be located /etc/pknw1/config and so set in the volume mappings
    you can also configure a container-config repo for backing up the persistent files
  
  
  
</details>
<hr>  

<details>
      <summary>Install and base configure Nginx Proxy Manager</summary>
      
  ```
    mkdir -p /etc/pknw1/docker/core-services/
    mkdir -p /etc/pknw1/config/core-services-config
    git init --separate-git-dir /etc/pknw1/github/core-services-config.git

    not we dont bother setting up a repo for just a compose file - only if it has more bits to keep togwther
  ```
  
  ```
        services:
          proxymanager:
            image: jc21/nginx-proxy-manager:latest
            restart: unless-stopped
            ports:
              - 100.100.69.2:80:80
              - 100.100.69.2:443:443
              - 100.100.69.2:81:81
              - 172.22.20.1:80:80
            privileged: true
            volumes:
              - /etc/pknw1/config/nginx_proxy_manager/98-themepark:/etc/cont-init.d/99-themepark
              - /etc/pknw1/config/nginx_proxy_manager/data:/data
              - /etc/pknw1/config/nginx_proxy_manager/letsencrypt:/etc/letsencrypt
            labels:
              - "com.centurylinklabs.watchtower.enable=true"
            container_name: proxymanager
            hostname: proxymanager
            networks:
              - proxy
              - admin
            environment:
              - VIRTUAL_HOST=proxymanager.admin.pknw1.co.uk
              - VIRTUAL_PORT=81
              - VIRTUAL_PROTO=http
  ```    
  
  this setup exposes an unencrypted port on your public IP address - so it is preferred if you connect via the local address assigned when connected to the admin network
  
  <i class="fa-regular fa-square-check"></i>login and configure your admin user<br>
  <i class="fa-regular fa-square-check"></i>docker compose down and docker compose up to check persistence<br>
  <i class="fa-regular fa-square-check"></i>once verified remove port 81 mappong from the docker compose<br>
  
  <i class="fa-regular fa-square-check"></i> restart NPM and prepare out inbound proxy hosts<br>
  
</details>
<hr>  


<details>
      <summary>Install internal Nginx Proxy Manager Split Networks Configuration</summary>
  
      your DNS is configured so that any hostname containing 'admin' resolves to your Tailscale Address
      you will open a service and if you are on VPN you will connect to any address for any service
      if you are not on Tailscale VPN, you will resolve to the Tailscale IP still, but have no access to it
      
      as we are only using one appliance for both "secured zones" there may be attempts to send a request for an admin
      service via the public IP; if they should bypass header checking, we also have an nginx rule ensuring that only users 
      on the 172 networks or within the TailNet are allowed access - any failed attmpts rewdirect through to public ip
      1. add the host - be sure to add the top level domain and a wildcard
      2. select SSL tab and create new cert request
      3. request a wildcard cert using DNS challenge
      4. setup the advamced rules checking source IP
  
      ```
        location ~* ^/$ {
            allow 100.100.69.0/24;
            allow 172.22.0.0/16;
            deny all;
          }
      you must ensure that the 172 range that you allow through here is only the admin range and not the proxy range
      ```
      
      ![](https://i.imgur.com/a4zoTSp.png)
    
</details>

<hr>  

<details>
      <summary>Install internal jwilder-nginx-proxy </summary>
      info
</details>
<hr>  

<details>
      <summary>Setup Dockge container management </summary>
      info
</details>


<hr>  
