Usage
----------

To configure this image, create a `config` directory, with a `script.pre` file inside it.  This
file should be marked as executable.  Place the following content in that file:

    #!/bin/bash
    
    echo "Generating configuration.."
    cat >/srv/www/config.inc.php <<EOF
    <?php
    /* vim: set expandtab sw=4 ts=4 sts=4: */
    /**
     * phpMyAdmin sample configuration, you can use it as base for
     * manual configuration. For easier setup you can use setup/
     *
     * All directives are explained in Documentation.html and on phpMyAdmin
     * wiki <http://wiki.phpmyadmin.net>.
     *
     * @version \$Id$
     * @package phpMyAdmin
     */
    
    /*
     * This is needed for cookie based authentication to encrypt password in
     * cookie
     */
    \$cfg['blowfish_secret'] = '<replace this with random text>';
    
    /*
     * Servers configuration
     */
    \$i = 0;
    
    /*
     * First server
     */
    \$i++;
    /* Authentication type */
    \$cfg['Servers'][\$i]['auth_type'] = 'cookie';
    /* Server parameters */
    \$cfg['Servers'][\$i]['host'] = '${LINKED_MARIADB_PORT_3306_TCP_ADDR}:${LINKED_MARIADB_PORT_3306_TCP_PORT}';
    \$cfg['Servers'][\$i]['connect_type'] = 'tcp';
    \$cfg['Servers'][\$i]['compress'] = false;
    /* Select mysqli if your server has it */
    \$cfg['Servers'][\$i]['extension'] = 'mysqli';
    \$cfg['Servers'][\$i]['AllowNoPassword'] = false;
    
    /* rajk - for blobstreaming */
    \$cfg['Servers'][\$i]['bs_garbage_threshold'] = 50;
    \$cfg['Servers'][\$i]['bs_repository_threshold'] = '32M';
    \$cfg['Servers'][\$i]['bs_temp_blob_timeout'] = 600;
    \$cfg['Servers'][\$i]['bs_temp_log_threshold'] = '32M';
    
    \$cfg['UploadDir'] = '';
    \$cfg['SaveDir'] = '';
    
    
    EOF

Note that the in the example above, we are linking to another MariaDB container.  You can replace the MariaDB host configuration with an explicit address if it's an external server.

To run this image:

    /usr/bin/docker run -v /path/to/config:/config --name=phpmyadmin --link mariadb:linked_mariadb hachque/phpmyadmin

To run this image when connecting to an external server, omit the `--link`:

    /usr/bin/docker run -v /path/to/config:/config --name=phpmyadmin hachque/phpmyadmin

What do these parameters do?

    -v path/to/config:/config = map the configuration from the host to the container
    --name phpmyadmin = the name of the container
    --link mariadb:linked_mariadb = (optional) if you are running MariaDB in a Docker container
    hachque/phpmyadmin = the name of the image

This assumes that you are using a reverse proxy container (such as `hachque/nginx-autoproxy`) to route HTTP and HTTPS requests to the phpMyAdmin container.  If you are not, and you want to just expose the host's HTTP and HTTPS ports to phpMyAdmin directly, you can add the following options:

    -p 80:80 -p 443:443

This image is intended to be used in such a way that a new container is created each time it is started, instead of starting and stopping a pre-existing container from this image.  You should configure your service startup so that the container is stopped and removed each time.  A systemd configuration file may look like:

    [Unit]
    Description=phpmyadmin
    Requires=docker.service mariadb.service
    
    [Service]
    ExecStart=<command to start instance, see above>
    ExecStop=/usr/bin/docker stop phpmyadmin
    ExecStop=/usr/bin/docker rm phpmyadmin
    Restart=always
    RestartSec=5s
    
    [Install]
    WantedBy=multi-user.target

SSH / Login
--------------

**Username:** root

**Password:** linux

**Port:** 24

(Note that repository hosting for Phabricator is served on port 22)

