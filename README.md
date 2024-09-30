# Docker-install-in-Laravel-project


To deploy a Laravel application on Docker using MySQL, Nginx, and SSL, you must create a Docker setup using Docker Compose. Below is a step-by-step guide:

Prerequisites:
Docker installed on your system.
Docker Compose installed.
A basic Laravel project setup.

Example Project Structure
Here’s an example of how your project might be structured for clarity:
```bash
my_project/
├── laravel-project----------------------------------- # main project folder
├── mysql ---------------------------------------------# create folder
├── nginx ---------------------------------------------# create folder
│    ├── conf.d-- -------------------------------------# create folder
│         └── default.conf 
├── php -----------------------------------------------# create folder
│   └── www.conf   
├── Dockerfile
├── nginx.dockerfile
├── docker-compose.yml
 ```

