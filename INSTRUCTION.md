This guide shows how to run a MySQL container with a mounted volume and how to run a Django App container that connects to this database.

DB: app_db user: app_user
DB password: 1234

1. Preparation (one-time setup)

Create a Docker network (optional, but convenient):

docker network create todonet
2. Run MySQL container with a volume

Assuming you have an image mysql-local:1.0.0, run it like this (container name — mysql-local):

docker run -d \
  --name mysql-local \
  --network todonet \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=1234 \
  -e MYSQL_DATABASE=app_db \
  -e MYSQL_USER=app_user \
  -e MYSQL_PASSWORD=1234 \
  mysql-local:1.0.0

Wait a few seconds and check logs:

docker logs -f mysql-local

You will see a message when the database is ready to accept connections.

3. Get MySQL container IP (needed for Django config)

If you are required to specify an IP, get it with:

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mysql-local

Example output: 172.18.0.2

NOTE: Alternatively, you can use the container name mysql-local as HOST if containers are in the same Docker network — this is simpler and more reliable. But if IP is required, use the command above.

4. Update Django configuration (todolist/settings.py)

Open todolist/settings.py and in the DATABASES section set the HOST (replace with IP from step 3 or use mysql-local):

DATABASES = {
    'default': {
        'ENGINE': 'mysql.connector.django',
        'NAME': 'app_db',
        'USER': 'app_user',
        'PASSWORD': '1234',
        'HOST': '172.18.0.2',  # <- replace with IP or 'mysql-local'
        'PORT': '',
    }
}

Save the changes.

5. Build/push App image (example)

If you have a local image todoapp:2.0.0, tag it and push to Docker Hub:

docker tag todoapp:2.0.0 <your-dockerhub-username>/todoapp:2.0.0
docker push <your-dockerhub-username>/todoapp:2.0.0

Docker Hub link:

https://hub.docker.com/r/midandnight/todoapp/general
6. Run App container
Option A — if the image runs server on internal port 8080:
docker run -d \
  --name todoapp \
  --network todonet \
  -p 8000:8080 \
  <your-dockerhub-username>/todoapp:2.0.0

Open in browser:
http://localhost:8000

Option B — force run server on port 8000:
docker run -d \
  --name todoapp \
  --network todonet \
  -p 8000:8000 \
  --entrypoint python \
  <your-dockerhub-username>/todoapp:2.0.0 \
  manage.py runserver 0.0.0.0:8000

Open:
http://localhost:8000

If you used MySQL IP (not container name), make sure it hasn’t changed after container restart.
Best practice — use container name (mysql-local) instead of static IP.

7. Useful debug commands

Check App logs:

docker logs -f todoapp

Run migrations inside container:

docker exec -it todoapp python manage.py migrate

Check containers status:

docker ps -a
8. Browser access

After successful start, open:

http://localhost:8000
If you mapped 8000:8080 → use http://localhost:8000
If you mapped 8000:8000 → also http://localhost:8000