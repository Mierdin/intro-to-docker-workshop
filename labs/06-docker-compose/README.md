# Multi-Container Applications with Docker Compose

* Create local files for building a Wordpress application
* Launch Wordpress via Docker Compose
* Test external connectivity to Wordpress

## Assemble local files for Wordpress and Compose

We're going to use the provided [Wordpress compose example](https://docs.docker.com/compose/wordpress/) and get this working in our Swarm cluster.

First, let's download wordpress and get into the resulting directory:

```
curl https://wordpress.org/latest.tar.gz | tar -xvzf -
cd wordpress
```

Next, the compose file we'll write will require a Dockerfile in this repository (to build our Wordpress app container) so let's build that now. This is fairly simple: we only need a base php5 image, and we need to copy the wordpress files into the container at build time:

```
cat > Dockerfile << EOF
FROM orchardup/php5
ADD . /code
EOF
```

Next, we need to write a docker-compose.yml file. This file describes our two-container app - one web and one db. Note there is no "link" section like there used to be - docker will automatically build an overlay network when we run docker compose with a special flag (we'll see this shortly):

```
cat > docker-compose.yml << EOF
web:
  build: .
  command: php -S 0.0.0.0:8000 -t /code
  ports:
    - "8000:8000"
  volumes:
    - .:/code
db:
  image: orchardup/mysql
  environment:
    MYSQL_DATABASE: wordpress
  labels:
    - "affinity:container!=web"
EOF
```

Finally, we need to install a wp-config.php file so Wordpress knows how to get to our DB container. Note that we're using the name of the container (we saw in the last lab that Docker inserts these names into each container's /etc/hosts file):

```
cat > wp-config.php << EOF
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', '');
define('DB_HOST', "wordpress_db_1:3306");
define('DB_CHARSET', 'utf8');
define('DB_COLLATE', '');

define('AUTH_KEY',         'put your unique phrase here');
define('SECURE_AUTH_KEY',  'put your unique phrase here');
define('LOGGED_IN_KEY',    'put your unique phrase here');
define('NONCE_KEY',        'put your unique phrase here');
define('AUTH_SALT',        'put your unique phrase here');
define('SECURE_AUTH_SALT', 'put your unique phrase here');
define('LOGGED_IN_SALT',   'put your unique phrase here');
define('NONCE_SALT',       'put your unique phrase here');

\$table_prefix  = 'wp_';
define('WPLANG', '');
define('WP_DEBUG', false);

if ( !defined('ABSPATH') )
        define('ABSPATH', dirname(__FILE__) . '/');

require_once(ABSPATH . 'wp-settings.php');
EOF
#?>
```

## Launch Wordpress with Compose

Now that we have these files in place, we can tell compose to start our application:

```
docker-compose --x-networking up -d
```

> The --x-networking flag enables currently experimental integration between [Docker Networking and Compose](https://docs.docker.com/compose/networking/).

The above does a few things:
- Runs a "docker build" for us on the web container, since we had that instruction there
- Creates an overlay network for these containers to communicate with each other, without having to use port forwarding (all enabled by the --x-networking flag).
- Starts two containers once the image has been built (using the "run" arguments provided there), and attaches them to the overlay network

We can ask compose for a summary of running applications (this is very useful with multiple applications, especially when running against a Swarm):

```
docker-compose ps
```

We can see the overlay network that compose created for us by listing them:

```
docker network ls
```

In particular, we can see that docker automatically attached our containers to this overlay network

```
docker network inspect wordpress
```

## Test external access to Wordpress

Next, let's try to access wordpress from our outside machine (laptop). First, we need to see what host the web application is running on, and curl to that location:

```
docker ps   #look for our wordpress_web... container, and note the host it's running on
lynx http://$(docker-machine ip < cluster node from above >):8000/
```

> Also note from the ```docker ps``` output above that only the web application is exposed to the outside world. Because of our overlay network, the web and db containers are communicating over the overlay network. (And because of our anti-affinity label, they are on different machines)

If all was configured correctly, we should be seeing the famous 5-minute Wordpress install!

# Destroy Compose Lab

```
docker-compose stop
docker rm wordpress_db_1
docker rm wordpress_web_1
docker rmi wordpress_web
docker volume rm $(docker volume ls -qf dangling=true)
```

## Navigation

[Previous Lab - Docker Multi-Host Networking](https://github.com/Mierdin/intro-to-docker-workshop/tree/master/labs/05-multihost-networking)
