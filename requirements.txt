<div align="center">

  <img src="assets/logo.svg" alt="logo" width="200" height="auto" />
  <h1>lulu backend</h1>

  <p>
    lulubackend services are the backend portal for Smartwealth web, mobile & admin panels containing all related APIs and functionalities to run the apps.
  </p>

<!-- Badges -->
<p>
  <a href="https://github.com/NeoMENATech/lulubackend/graphs/contributors">
    <img src="https://badgen.net/static/contributers/30/orange" alt="contributors" />
  </a>
  <a href="https://github.com/NeoMENATech/lulubackend/releases">
    <img src="https://badgen.net/static/release/v2/blue" alt="release" />
  </a>
  <a href="https://github.com/NeoMENATech/lulubackend/actions">
    <img src="https://badgen.net/static/ci/passing/green" alt="ci" />
  </a>
</p>

<h4>
    <a href="https://api.nbkcapitalsmartwealth.com/api/v2">Documentation</a>
  <span> · </span>
    <a href="https://neotechnologies.atlassian.net/jira/software/c/projects/NEOS/boards/57">Report Bug</a>
  <span> · </span>
    <a href="https://neotechnologies.atlassian.net/jira/software/c/projects/NEOS/boards/57">Request Feature</a>
  </h4>
</div>

<br />

<!-- Table of Contents -->

### :notebook_with_decorative_cover: Table of Contents

- [Tech Stack](#space_invader-tech-stack)
- [Deploy Locally](#triangular_flag_on_post-deploy-locally)
- [Deploy on AWS](#cloud-deploy-on-aws)
- [API Documentation](#books-api-documentation)
- [Cronjobs Documentation](#notebook_with_decorative_cover-cronjobs-documentation)
- [Automatic Testing](#testing-commands)

<!-- TechStack -->

### :space_invader: Tech Stack

- Flask
- Docker
- PostgreSQL
- Redis
- Python
- HTML

<!-- Deploy Locally -->

### :triangular_flag_on_post: Deploy Locally

#### Build and run the image

- Make sure are in the project root dicectory **or** where _docker-compose.yml_ file is located

  ```
  cd setup-local/
  docker compose -f development.yml build
  ```

- After build process ends successfully, run the app

  ```
  docker compose -f development.yml up
  ```

- You can use **uv** by running

  ```
  cd setup-local/
  docker compose -f development-uv.yml build
  ```

- After build process ends successfully, run the app
  ```
  docker compose -f development-uv.yml up
  ```

#### Setup NGINX

- Check if default file exists

  ```
    ls /etc/nginx/sites-enabled/default
  ```

  - If no, create it with below content

    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```

    ```
    server {
      listen       80;
      server_name  neo-backend;
      location / {
          proxy_pass  http://localhost:8000/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_redirect off;
          proxy_buffering   off;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
      }
    }

    server {
      listen 80;
      server_name neo-db;
      location / {
          proxy_pass  http://localhost:5055/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_redirect off;
          proxy_buffering   off;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
      }
    }

    server {
      listen 80;
      server_name redis-insight;
      location / {
          proxy_pass  http://localhost:8011/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_redirect off;
          proxy_buffering   off;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
      }
    }

    server {
      listen 80;
      server_name analytics;
      location / {
          proxy_pass  http://localhost:2345/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
          proxy_redirect off;
          proxy_buffering   off;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
      }
    }

    ```

- Add the hostnames

  ```
  sudo nano /etc/hosts
  ```

  ```
  127.0.0.1       neo-backend
  127.0.0.1       neo-db
  127.0.0.1       redis-insight
  127.0.0.1       analytics
  ```

- Finally, restart nginx
  ```
  sudo systemctl restart nginx
  ```
- You can access the backend on `http://neo-backend/api/v2`
- you can access the db on `http://neo-db` (use `admin@admin.com` and `root` as user name and password)
- you can access the redis insight by hitting `http://redis-insight`
- you can access the grafana dashboard by hitting `http://analytics`

#### Setup Redis Insight

After accessing the redis insight `http://redis-insight` for the first time, **make sure to select only the agree for condition toggle button, and try to leave the other 4 toggle buttons disabled (for security purposes)**:
Steps:

- ADD REDIS DATABASE
- ADD the following configuration:

```python
  Host: lulubackend-redis
  Port: 6379
  Name: db-redis
  Username:
  Passowrd:
```

- Then ADD REDIS DATABASE
- After doing those steps, you can see the redis database name, add, update, or delete values from the insight, and even access the cli.

#### Analytics(grafana):

After accessing the grafana using `http://analytics`, you can go to the side bar to the dashboards section, and you'll find an application with the name **lulubackend monitoring**:

<p align="center">
  <img src="assets/grafana.png" alt="grafana" width="50%"/>
</p>

#### Datamart local setup:

1- On Dbeaver/PgAdmin add the datamart database connection
2- Build and run the lulubackend normally
3- For migration add --multidb for the flask db init command

#### Important commands for running the project locally

- To open an interactive shell with the container
  ```
  docker exec -it lulubackend bash
  ```
- To create a sequence id for user_profile table
  ```
  flask create_internal_id_seq
  ```
- Database migrations:

  - Remove migrations directory
    ```
    rm -rf migrations
    ```
  - Drop alembic table from database
    ```
    flask drop_alembic_table
    ```
  - Generate and apply migrations
    ```
    flask db init
    flask db migrate
    flask db upgrade
    ```
  - To clean your repo from new migrations folder and files (since we are migrating on other envs)
    ```
    sudo  git clean -f -q -- /home/alim91/Workspace/lulubackend/migrations/
    ```

- To import data from a backup file, you should follow the following process:

  - name_of_the_backup_file: name of the data backup file shared by the devOps team or any developer.

  ```
  docker cp {name_of_the_backup_file} lulubackend-postgres:.
  ```

  ```
  docker exec lulubackend-postgres pg_restore --host "localhost" --port "5432" --username user --password --dbname "lulubackend_db" --verbose "/{name_of_the_backup_file}"
  ```

  :warning: The username and the database name are not the same on every machine, so please check yours.

- To run celery queues (run all or some of them)
  ```
  flask celeryworker_synchronization
  flask celeryworker_mailing
  flask celeryworker_data_migration
  flask celeryworker_continuous
  flask celeryworker_high_priority
  ```
- To run test
  ```
  flask run_test
  ```

### :cloud: Deploy on AWS

#### Adding VPN

- As we know, we have restrictions on accessing any of our AWS services. So, you should contact the devOps team to get your **config** file.
- You can check the following VPN documentation:
  https://neotechnologies.atlassian.net/wiki/spaces/DevOps/pages/1549795386/VPN+Connection+Setup+-+AlgoVPN

#### Deploy on ECS

- Go to https://github.com/NeoMENATech/lulubackend/actions/workflows/ecsdeploy.yaml
- Deploy with below options:

  - From branch directly
  - With a pull request

- To get ssh access to lulubackend container
  - Configure AWS CLI https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html
  - Execute the following command to check the instances:
    ```
    aws ecs list-tasks --cluster lulubackend-cluster-uat --service-name lulubackend-service-uat --region eu-central-1
    ```
  - Execute below command
    ```
    aws ecs execute-command  \
    --region eu-central-1 \
    --cluster lulubackend-cluster-uat \
    --task 46b9837e4da24f149d364fe435df5721 \
    --container lulubackend \
    --command "/bin/bash" \
    --interactive
    ```

#### Access UAT on AWS via SSH

- Activate the wg-quick@wg0 VPN.
- Configure the AWS CLI session manager https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html
- Go to https://neomenatech.awsapps.com/start#/
- Go to the AWS Account box, and you'll see a user with **neo-unprivleged-devs** name.
- Click on **Command line or programmatic access** and export the 3 tokens to your system.
- Then try to run on a cmd the following command:
  ```
  aws ecs list-tasks --cluster lulubackend-cluster-uat --service-name lulubackend-service-uat
  ```
- You will get the instances.
- Then, if you want to access the UAT instance, you should replace the **instance_hashkey** and run the following:
  ```
  aws ecs execute-command  \
    --region eu-central-1 \
    --cluster lulubackend-cluster-uat \
    --task {task_id} \
    --container lulubackend \
    --command "/bin/bash" \
    --interactive
  ```

#### S3 UAT Access

You can get the file from the S3 bucket by running the following command:

```
aws s3 cp s3://[bucket-name]/[file-path] [output]

aws s3 cp s3://example/demo/b12dd666-46fd-47dc-96ae-6c594fa69d57-signed.pdf doc.pdf
```

### Testing Commands

This project includes a set of commands to simplify running and managing tests in a Dockerized environment.
Please refer to the documentation for more information: https://neotechnologies.atlassian.net/wiki/spaces/~5b1122d281c3df12acf53346/pages/2234875943/Initiative+API+Module+Reliability+and+Testing+Enhancement

### :notebook_with_decorative_cover: Cronjobs Documentation

- https://neotechnologies.atlassian.net/wiki/spaces/BT/pages/1452507140/NBK+Cronjobs
