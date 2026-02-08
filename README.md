# CiviCRM Docker

This repository contains resources to run CiviCRM on Docker.

Container images are published to [Docker Hub](https://hub.docker.com/u/civicrm) for all stable versions of CiviCRM *standalone* as part of CiviCRM's regular [release process](https://docs.civicrm.org/dev/en/latest/core/release-process/).

If you are looking for a **ready to use** CiviCRM application, use `civicrm/civicrm`. If you are looking for an image that you can use as part of a customised **Docker build process**, use `civicrm/civicrm-base`.

Note: WordPress images are fully implemented but not yet published to Docker Hub. Other CMS integrations (Joomla, Backdrop, Drupal) are not currently available.

## Quick start

Note: these instructions are not designed for use in a production set up - they are intended to provide a minimal local environment for testing purposes. They assume you are comfortable working with docker and docker compose. If that's not the case, then see the resources below for a quick introduction:

- https://docs.docker.com/get-started/
- https://docs.docker.com/compose/gettingstarted

### Running the image

Run the CiviCRM image with. `docker run --detach --publish 8000:80 civicrm/civicrm`. You'll see CiviCRM's installation screen at http://localhost:8000 where you will be prompted for database credentials, etc. 

### With docker compose

A more complete 'quick start' built with docker compose can be found in the [`example`](example) directory.

1. clone this repository `git clone https://github.com/civicrm/civicrm-docker`
2. change into the example directory `cd civicrm-docker/example/civicrm`
3. create an `.env` file with two environment variables:
```shell
# .env
MYSQL_PASSWORD=INSECURE_PASSWORD        # change these to
MYSQL_ROOT_PASSWORD=INSECURE_PASSWORD   # if you want to
```
4. start the compose project with `docker compose up -d`
5. wait for the database to initialise (you can check progress with `docker compose logs -f`).
6. install CiviCRM with `docker compose exec -u www-data -e CIVICRM_ADMIN_USER=admin -e CIVICRM_ADMIN_PASS=password app civicrm-docker-install` (note that we are passing in the admin username and password as environment variables here - you can change them if you want to).
7. visit http://localhost:8760 and log in using the credential supplied above.
8. when you are finished, bring the project down with `docker compose down`.

## Environment variables

At a minimum, you should set the following environment variables:

- `CIVICRM_DB_HOST`
- `CIVICRM_DB_PORT`
- `CIVICRM_DB_NAME`
- `CIVICRM_DB_USER`
- `CIVICRM_DB_PASSWORD`
- `CIVICRM_UF_BASEURL`

Note that the `CIVICRM_DB_*` can be replaced with a single `CIVICRM_DSN` variable.

**Experimental**: you can override the default apache port (in the container) by setting `APACHE_PORT`.

## Installation


The `civicrm/civicrm` image comes with a convenience script for installing a site: `civicrm-docker-install`. The script expects database credentials and the admin username (`CIVICRM_ADMIN_USER`) and password (`CIVICRM_ADMIN_PASS`) to be set as environment variables.

It calls the standard CiviCRM installation process. See [build/civicrm/civicrm-docker-install](build/civicrm/civicrm-docker-install) for more details and the docker compose instructions above for an example of how you might call this script.

See also https://docs.civicrm.org/installation/en/latest/standalone/ for more details on the CiviCRM Standalone installation.

## Volumes

The `/var/www/html/public`, `/var/www/html/private` and `/var/www/html/ext` directories should be persisted. See the [`example/civicrm/compose.yaml`](example/civicrm/compose.yaml) file for an example.

## Tags

You can use tags to specify a CiviCRM version and php version, for example:

`civicrm/civicrm:6.0-php8.3`

### CiviCRM version

Keep up to date with the latest stable '5.x' release by using the tag `5`, which will receive all minor and patch releases. Pin your site to a minor release by using a minor version tag. For example, `6.0` will receive all patch releases for the 6.0 minor version. 
Skip the tag to default to the latest stable release.

### PHP version

Images are published for all supported versions of PHP. Specify a php version with a tag like `php8.3`.

Skip the tag to default to the [the most recent version recommended by CiviCRM](https://docs.civicrm.org/installation/en/latest/general/requirements/#php-version).

### Extended support release

**WORK IN PROGRESS**

Subscribers to the ESR should soon be able to download images for the ESR from a private registry on https://lab.civicrm.org.

## Building images

If you have specific needs that are not catered for by the pre-built images that are published on Docker Hub, you may want to build an image locally using the Dockerfiles in the `build` directory.

### The `build/civicrm` directory

The `build/civicrm` Dockerfile is suitable for the most straight forward deployments. You must pass one of either `CIVICRM_VERSION` or `CIVICRM_DOWNLOAD_URL` and the `PHP_VERSION` as build arguments:

- `CIVICRM_VERSION` specifies a (stable) CiviCRM version
- `CIVICRM_DOWNLOAD_URL` specifies the tarball to download. Useful to build [release candidates and nightly releases](https://download.civicrm.org/latest/). This argument overrides `CIVICRM_VERSION`.
- `PHP_VERSION` specifies the PHP version. Useful if you want to build using a PHP version that we are not building images for.

For example:

Build an image using CiviCRM 6.0 and PHP version 8.3:

```shell
docker build build/civicrm --build-arg CIVICRM_VERSION=6.0 --build-arg PHP_VERSION=8.3 -t my-custom-build
```

Build an image with the latest nightly version of CiviCRM:

```shell
docker build build/civicrm --build-arg CIVICRM_DOWNLOAD_URL=https://download.civicrm.org/latest/civicrm-NIGHTLY-standalone.tar.gz --build-arg PHP_VERSION=8.3 -t my-civi/civicrm
```

Build an image with the latest nightly version of CiviCRM and a specific release of PHP. In this case, we'll need to build the intermediary images.

The `build.php` can help with this:

```shell
./build.php --php-version=8.3 --image-prefix=my-civi --skip-push
```

If you run `docker image ls "my-civi/*"` after this, you will see something like this: 

```
REPOSITORY             TAG            IMAGE ID       CREATED           SIZE
my-civi/civicrm        6              91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        6-php8.3       91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        6.0            91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        6.0-php8.3     91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        6.0.3          91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        6.0.3-php8.3   91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        latest         91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm        php8.3         91d9a3048d81   1 minutes ago     694MB
my-civi/civicrm-base   latest         29f972ae8200   3 minutes ago     561MB
my-civi/civicrm-base   php8.3         29f972ae8200   3 minutes ago     561MB
my-civi/common-base    latest         29f972ae8200   7 minutes ago     561MB
my-civi/common-base    php8.3         29f972ae8200   7 minutes ago     561MB
```

### Custom builds

If you have a custom build process, for example if you have a special way to download CiviCRM, or want to install CiviCRM extensions in the image, consider using `civicrm/civicrm-base` as your base image.

For example:

```Dockerfile
FROM civicrm/civicrm-standalone-base:php8.3

RUN curl https://whizzy.com/download/whizzy.tar.gz && \
  tar -xf whizzy.tar.gz 
```

### Image architecture

```mermaid
flowchart BT
    C[civicrm]
    B[civicrm-base] --> C
    A[common-base] --> B
    E[wordpress]
    D[wordpress-base] --> E
    A --> D
```

Note: WordPress images are fully implemented but not yet published to Docker Hub.

## Management

The `./build.php` script can be used to build images.

Calling `./build.php` without any arguments will build the latest stable version of CiviCRM and push it to docker hub.

If you are publishing official images on Docker Hub, make sure to run it in an environment that can publish multiplatform images, and can push to the CiviCRM docker account.

Command options are as follows:

- **--image-prefix=** - a custom prefix for generated images (defaults to `civicrm`)
- **--image-filter=** - only build the specified images (comma seperated list)
- **--php-version=** - build a single specific php version (defaults to all supported versions)
- **--download-url=** - a specific tarball to download  
- **--builder=** - the docker build builder to use
- **--platform=** - the platforms to build for
- **--skip-push** - build the images but do not push them to Docker Hub
- **--no-cache** - do not use a cache when building the images
- **--dry-run** - just output the commands that would be executed
- **--step** - run one step at a time

Note: before running `./build.php`, you will need to install the required dependencies with `composer install` (see https://getcomposer.org/ for more details).

## WordPress

The WordPress + CiviCRM images provide CiviCRM running as a WordPress plugin. These images are fully implemented but not yet published to Docker Hub.

If you are looking for a **ready to use** WordPress + CiviCRM application, use `civicrm/wordpress`. If you are building a custom image, use `civicrm/wordpress-base`.

### Quick start

**Note**: These instructions are for testing purposes, not production deployment.

#### Running the image

```shell
docker run --detach --publish 8000:80 civicrm/wordpress
```

You must complete the installation process (see below) before WordPress and CiviCRM are usable.

#### With docker compose

A complete example is in the [`example/wordpress`](example/wordpress) directory.

1. Clone this repository: `git clone https://github.com/civicrm/civicrm-docker`
2. Change directory: `cd civicrm-docker/example/wordpress`
3. Review/edit `.env` file (contains passwords and admin credentials)
4. Start containers: `docker compose up -d`
5. Wait for database initialization: `docker compose logs -f db` (wait for "ready for connections")
6. Install WordPress and CiviCRM: `docker compose exec -u www-data app civicrm-docker-install`
7. Visit http://localhost:8760 and log in with credentials from `.env`
8. When finished: `docker compose down`

### Deployment Workflows

WordPress sites have different requirements for production vs development. Choose the workflow that matches your needs:

#### Workflow A: Simple Production (Immutable Infrastructure)

**Best for**: Locked-down CiviCRM-only sites where plugins are managed via Docker images.

**Configuration**:
- CiviCRM is baked into the Docker image
- Mount only: `/var/www/html/wp-content/uploads` (user uploads)
- No additional plugin management via WordPress admin

**Limitations**: Plugins installed via WordPress admin will be lost on container restart.

**Example** (current `compose.yaml` default):
```yaml
volumes:
  - uploads:/var/www/html/wp-content/uploads
  - private:/var/www/private
```

**Installation**: Use `civicrm-docker-install` script (installs CiviCRM from image).

#### Workflow B: Flexible Production (Persistent wp-content)

**Best for**: Production sites where administrators need to install/update plugins and themes via WordPress admin.

**Configuration**:
- Mount entire `/var/www/html/wp-content` directory
- CiviCRM is NOT baked into image - install manually into mounted volume
- Full plugin/theme management via WordPress admin

**Trade-offs**: Larger volumes, must manage CiviCRM updates manually.

**Example**:
```yaml
volumes:
  - wpcontent:/var/www/html/wp-content
  - private:/var/www/private

volumes:
  wpcontent:
  private:
```

**Installation**:
1. Start containers (without CiviCRM installed yet)
2. Install WordPress core: `docker compose exec -u www-data app wp core install ...`
3. Download CiviCRM to mounted volume: `docker compose exec -u www-data app wp plugin install https://download.civicrm.org/civicrm-VERSION-wordpress.zip`
4. Activate and configure CiviCRM via WordPress admin

**Note**: The `civicrm-docker-install` script expects CiviCRM in the image, so it won't work for this workflow. Use manual installation instead.

#### Workflow C: Development (Local Code Override)

**Best for**: CiviCRM core developers or plugin developers who need to work on local code.

**Configuration**:
- CiviCRM from image is overridden by bind mount to local directory
- Mount local CiviCRM code at `/var/www/html/wp-content/plugins/civicrm`
- Changes to local files appear immediately in container

**Example** (`compose.dev.yaml`):
```yaml
services:
  app:
    volumes:
      - /path/to/local/civicrm-wordpress:/var/www/html/wp-content/plugins/civicrm
```

**Usage**:
```shell
# Start with development override
docker compose -f compose.yaml -f compose.dev.yaml up -d

# Make changes to local code - they appear instantly

# Switch back to production (image-based code)
docker compose down
docker compose up -d
```

**Installation**: Use `civicrm-docker-install` script (works with mounted code).

**See**: [`example/wordpress/compose.dev.yaml`](example/wordpress/compose.dev.yaml) for complete example.

### Environment variables

**CiviCRM database**:
- `CIVICRM_DB_HOST`, `CIVICRM_DB_PORT`, `CIVICRM_DB_NAME`, `CIVICRM_DB_USER`, `CIVICRM_DB_PASSWORD`
- OR `CIVICRM_DSN` (e.g., `mysql://user:pass@host:3306/database`)

**WordPress database**:
- `WORDPRESS_DB_HOST` - Database host (e.g., `db`)
- `WORDPRESS_DB_NAME` - Database name (typically same as CiviCRM database)
- `WORDPRESS_DB_USER` - Database user
- `WORDPRESS_DB_PASSWORD` - Database password
- `WORDPRESS_CONFIG_FILE` - Path to store wp-config.php (e.g., `/var/www/private/wp-config.php`)

**Site configuration**:
- `CIVICRM_UF_BASEURL` - Site URL (e.g., `http://localhost:8760`)
- `WORDPRESS_SITE_TITLE` - WordPress site title

**Installation credentials**:
- `CIVICRM_ADMIN_USER` - Admin username
- `CIVICRM_ADMIN_PASS` - Admin password
- `CIVICRM_ADMIN_EMAIL` - Admin email

**Optional**:
- `APACHE_PORT` - Override Apache port inside container (default: 80)
- `PHP_MEMORY_LIMIT` - PHP memory limit (default: 256M)

### Installation

#### For Workflows A and C (civicrm-docker-install)

The `civicrm/wordpress` image includes `civicrm-docker-install` script that:
1. Creates wp-config.php at `WORDPRESS_CONFIG_FILE` location
2. Installs WordPress core
3. Installs CiviCRM plugin

**Prerequisites**: All environment variables must be set.

**Usage**:
```shell
docker compose exec -u www-data app civicrm-docker-install
```

**Important**: Run as `www-data` user to ensure correct file permissions.

See [build/wordpress/civicrm-docker-install](build/wordpress/civicrm-docker-install) for details.

#### For Workflow B (Manual Installation)

When using persistent wp-content volume:

1. **Install WordPress core**:
   ```shell
   docker compose exec -u www-data app wp core install \
     --url=${CIVICRM_UF_BASEURL} \
     --title="${WORDPRESS_SITE_TITLE}" \
     --admin_user=${CIVICRM_ADMIN_USER} \
     --admin_password=${CIVICRM_ADMIN_PASS} \
     --admin_email=${CIVICRM_ADMIN_EMAIL}
   ```

2. **Download CiviCRM**:
   ```shell
   docker compose exec -u www-data app wp plugin install \
     https://download.civicrm.org/civicrm-6.0-wordpress.zip
   ```

3. **Activate and configure** CiviCRM via WordPress admin interface.

### Volumes

Volume requirements depend on your deployment workflow:

#### Workflow A (Simple Production):
```yaml
volumes:
  - uploads:/var/www/html/wp-content/uploads    # User uploads (required)
  - private:/var/www/private                     # WordPress config (required)
```

#### Workflow B (Flexible Production):
```yaml
volumes:
  - wpcontent:/var/www/html/wp-content  # Entire wp-content (includes plugins, themes, uploads)
  - private:/var/www/private             # WordPress config (required)
```

#### Workflow C (Development):
```yaml
volumes:
  - uploads:/var/www/html/wp-content/uploads              # User uploads
  - private:/var/www/private                               # WordPress config
  - /local/path:/var/www/html/wp-content/plugins/civicrm  # Local CiviCRM code
```

### Tags

WordPress images use the same tagging strategy as CiviCRM Standalone:

- `civicrm/wordpress:latest` - Latest stable CiviCRM + recommended PHP
- `civicrm/wordpress:6` - Latest CiviCRM 6.x + recommended PHP
- `civicrm/wordpress:6.0` - Latest CiviCRM 6.0.x + recommended PHP
- `civicrm/wordpress:6.0-php8.3` - CiviCRM 6.0.x with PHP 8.3
- `civicrm/wordpress:php8.3` - Latest stable CiviCRM with PHP 8.3

### Building images

Build WordPress images locally using the Dockerfile:

```shell
docker build build/wordpress \
  --build-arg WORDPRESS_VERSION=6.9 \
  --build-arg CIVICRM_VERSION=6.0 \
  --build-arg PHP_VERSION=8.3 \
  -t my-org/wordpress-civicrm
```

Or use the build script:

```shell
# Install dependencies first
composer install

# Build WordPress images
./build.php --image-filter=wordpress-base,wordpress --php-version=8.3 --skip-push
```

This builds the full hierarchy: `common-base` → `wordpress-base` → `wordpress`.

#### Custom builds

For custom WordPress + CiviCRM images, extend `civicrm/wordpress-base`:

```Dockerfile
FROM civicrm/wordpress-base:php8.3

# Install additional WordPress plugins
RUN wp plugin install custom-plugin --activate --allow-root

# Download specific CiviCRM version
ARG CIVICRM_VERSION=6.0.3
RUN curl -L https://download.civicrm.org/civicrm-${CIVICRM_VERSION}-wordpress.zip \
  -o /tmp/civicrm.zip && \
  unzip /tmp/civicrm.zip -d /var/www/html/wp-content/plugins && \
  rm /tmp/civicrm.zip
```