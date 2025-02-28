## Configuring Core Server Components
- [x] Networking
- [x] VPN 
- [x] Docker
- [x] Docker Ecosystem


### Docker Core System
With docker installed on the system we can now setup the admin and proxy networks, and our critical systems

<details>
  <summary>Networking Setup</summary>

After docker is installed, we will create our external networks and assign ranges

| ```docker network create --subnet 172.22.22.0/24 admin ``` | ```docker network create --subnet 172.22.21.0/24 proxy ```|
| -- | -- |
 
</details>


Once docker is installed, we need to put the standards, templates and mechanisms for running a complex docker stack; 

| # | Description |
| -- | -- |
| 1 | Setup for BAU activity |
| 2 | Setup the split-tier networking and VPN Access|
| 3 | Implement our "core-services" in docker; setup backups, automation etc |


