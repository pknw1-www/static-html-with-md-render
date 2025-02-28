## Configuring Core Server Components
- [x] Networking
- [x] VPN 
- [x] Docker
- [x] Docker Ecosystem


### Docker Ecosystem
Once docker is installed, we need to put the standards, templates and mechanisms for running a complex docker stack; 

| # | Description |
| - | - |
| 1 | Enforce a standard filesystem layout and templates/scripts for BAU operation; managing repositories, configurations, persistent data storage |
| 2 | Setup the split-tier networking model where we can expose ports on the public IP, but all admin proxying will take place via the Tailscale VPN |
| 3 | Implement our "core-services" in docker; setup backups, automation etc |


