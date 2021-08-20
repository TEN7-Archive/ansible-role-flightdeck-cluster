# Getting started for local development

Flightdeck has been extensively tested on macOS and Linux, and should work on Windows without issue.

## Installing Docker

Flightdeck is based on Docker. Docker allows you to spin up multiple, lightweight virtual environments on your system called "containers". You can find out more about containers on [training.docker.org](https://training.docker.com/introduction-to-docker).

For macOS and Windows:
1. [Download](https://www.docker.com) Docker for macOS or Docker for Windows.
2. Run the installer and follow on-screen instructions.
3. Restart your computer when finished.

For Linux:
* Consult your distribution's documentation for the proper installation method.

## Adding Flightdeck to your project

When using Flightdeck, it's best to add it permanently to your project repository. That way, anyone working on that project has access to the some stack of containers.

1. Reorganize your repository such that the site docroot is in a **subdirectory** of your project root. This is assumed to be `src` by default:
```
/path/to/my_project
├── .git/
└── src/
    ├── core/
    └── index.php
```
2. Using the [sample project](https://github.com/ten7/flightdeck.ten7.io/tree/main/sample-project), add `db-backups/.gitignore` to your project.
```
/path/to/my_project
├── .git/
├── db-backups/
│   └── .gitignore
└── src/
    ├── core/
    └── index.php
```
4. Using the [sample project](https://github.com/ten7/flightdeck.ten7.io/tree/main/sample-project), copy the `docker-compose.yml`, `docker-compose.override.yml.example`, and `flight-deck.yml` into the root of your project.
```
/path/to/my_project
├── .git/
├── db-backups/
│   └── .gitignore
├── docker-compose.yml
├── docker-compose.override.yml.example
├── flight-deck.yml
└── src/
    ├── core/
    └── index.php
```
5. If using Solr, create the `solr-conf/yourSolrVersion` to the root of your project to hold Solr core configs:
```
/path/to/my_project
├── .git/
├── db-backups/
│   └── .gitignore
├── docker-compose.yml
├── docker-compose.override.yml.example
├── flight-deck.yml
├── solr-conf
│   └── 8.x
└── src/
    ├── core/
    └── index.php
```
6. For Drupal sites, create the `config/` directory:
```
/path/to/my_project
├── .git/
├── db-backups/
│   └── .gitignore
├── config/
│   └── .gitkeep
├── docker-compose.yml
├── docker-compose.override.yml.example
├── flight-deck.yml
└── src/
    ├── core/
    └── index.php
```

## Using Flightdeck outside your site repo

Some hosting providers such as Pantheon, prefer that your site docroot and repository root are the same directory. In those cases, you cannot install Flightdeck as described above. In those cases, you can use Flightdeck as a separate project.

1. Set up the directory structure as laid out above, minus the `src` directory.
2. Clone your site repo into a directory named `source` in your Flightdeck directory.
3. Alternatively, create a symlink named `src` in your Flightdeck directory to your site repository.
4. Add `src` to the Flightdeck `.gitignore`, if persisting it to a repository.

## Database connection

As a set of Docker containers, the database connection information for your site is somewhat different than you may expect. While all the containers are running on your host OS and are accessible via `localhost`, the site sees itself on a server named `web`, and the database is on another server named `db`. As a result, we need to configure the site to access the database remotely.

### Configuration for Drupal

If Drupal is already installed:
1. Use your text editor of choice to open you `settings.php` or `settings.local.php` file.
2. Update the $database variable to the following. This will instruct Drupal to use the MySQL login specified in the `.env` file:
```php
$databases['default']['default'] = array(
  'database' => getenv('MYSQL_NAME'),
  'username' => getenv('MYSQL_USER'),
  'password' => getenv('MYSQL_PASS'),
  'host' => 'db',
  'port' => '',
  'driver' => 'mysql',
  'prefix' => '',
 );
```
3. Save the file.

If Drupal is not installed:
1. Open the `docker-compose.yml` file. Note the values of `MYSQL_NAME`, `MYSQL_USER`, and `MYSQL_PASS`.
2. Start Flightdeck using `docker-compose`:
```shell
docker-compose up -d
```
3. Begin the installation process as normal.
4. On the **Database configuration** page, enter the **Database name**, **username**, and **password** as specified in the `docker-compose.yml` file.
5. Open **Advanced Options**. For the **Host** enter `db`.
6. Continue with the installation.

## Create a URL alias

Accessing Flightdeck using `localhost` poses a number of problems. The biggest of which is that it's a special domain name, which creates problems for complex CMSes like Drupal. Many sites also rely on `.htaccess` rewrite rules to redirect traffic to an HTTPS or `www.`-prefixed domain.

To solve these issues, it's ***highly*** recommended to create a *URL alias*.

On Linux and macOS:

1. Open a terminal emulator.
2. Open the `/etc/hosts` file using your favorite text editor. Be sure to use `sudo` to run the command as the root user: `sudo vi /etc/hosts`
3. Notice there are two columns of text, one is IP address, the other is host names.
4. At the end of the file, enter a new row. Use `127.0.0.1` as the IP address, and `docker.test` as the hostname:
```
127.0.0.1 docker.test
```
5. Check the file for typos, particularly "docker.te**x**t".

On Windows:

1. Using the Start Menu, locate the **Notepad** application.
2. Right click **Notepad**, and select **Run as administrator**.
3. Open the following file for editing: `C:\Windows\System32\Drivers\etc\hosts`
4. Notice there are two columns of text, one is IP address, the other is host names.
5. At the end of the file, enter a new row. Use `127.0.0.1` as the IP address, and `docker.test` as the hostname:
```
127.0.0.1 docker.test
```
5. Check the file for typos, particularly "docker.te**x**t".

You only need to create the `docker.test` alias once, no matter how many Flightdeck-powered projects you have on your system. You may also choose to add a different URL alias such as 'docker.test', '[your_site_name].test', and so on.

### Using Flightdeck

See [using](using.md).

### XDebug configuration

See [xdebug](xdebug.md).
