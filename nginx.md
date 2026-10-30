updated 

sudo apt update

![sudo apt update](images/sudoupdate.png)

Ngnix installagtion

![Nginx installation](images/nginx.png)

status

![Nginx installation](images/nginx-status.png)

Test web server

![Nginx installation](images/Test-web-server.png)

run URL in host browser

![Nginx installation](images/browser-nginx.png)


Architectural diagram

![Nginx installation](images/architecture.jpeg)

giving permission to folder of nginx 

![Nginx installation](images/permission.png)

adding html in ngnix 

![Nginx installation](images/adding-html.png)

Configure Nginx to serve your website

![Nginx installation](images/index.html.png)

### What did we configure?

| Directive | Purpose |
|-----------|---------|
| `listen 80` | Listen for HTTP requests |
| `root` | Website directory |
| `index` | Default homepage |
| `server_name` | Matches incoming hostname |
| `location /` | Handles website requests |
| `try_files` | Checks whether requested files exist |

check the configuration file status

![Nginx installation](images/status-of-nginx.png)

reloading the bash

![Nginx installation](images/reload-nginx.png)
