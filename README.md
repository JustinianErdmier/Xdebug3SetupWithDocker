# Xdebug 3 Setup with Docker
While I use PhpStorm as my IDE of choice, this setup process should work with any PHP IDE that supports advanced debugging with Xdebug. Additionally, this can also be setup without Docker, just ensure you install Xdebug locally on your machine.

This setup process is how I was able to get Xdebug 3 working in my projects after watching [Programming with Gio's short tutorial](https://youtu.be/7YuYxbYd3P0). Even using this setup, I highly recommend watching his video.

## Step 1: Installing Xdebug
Again, this setup is for Docker. If you are not using Docker, you can install Xdebug on your local machine. Just following the [install directions here](https://xdebug.org/wizard). Additional information about Xdebug can be found [here on there Github](https://github.com/xdebug/xdebug#xdebug).

In your Dockerfile, add the line to install Xdebug. For me, it looks like the following:
```
RUN apt-get update; \
    apt-get -y --no-install-recommends install \
        php8.1-gd \
        php8.1-mysql \
        php8.1-xdebug; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*
```
But it might also look like this:
```
RUN pecl install xdebug \
    && docker-php-ext-enable xdebug
```
> If you install Xdebug like the latter, then you will want to exclude or comment out the ***zend_extensions=*** property in your **php.ini** file. We'll revisit this again in Step 3.

Before you finalize the installation and rebuild your container, if you use docker-compose, jump to Step 1.5 first, and then come back.

Final step is to rebuild the container. In the Terminal, navigate to the directory of your **docker-compose.yml** file, and execute `docker up -d --build`. This will safely rebuild your containers to include any changes you made, without wiping any data (i.e., it won't delete your databases if you have a SQL container).

### Step 1.5: Docker Compose
Ensure your **docker-compose.yml** file is correct. If you added the property ***ports:*** to your **php-fpm** section, go ahead and remove it. Xdebug does not require any special configuration within the **docker-compose.yml** file to work.

Reference the **docker-compose.yml** file included in this repo. If yours looks good, go back to Step 1 and finish the installation process.

## Step 2: Configuring the Web Server
For this process, we're using nginx.

In your **nginx.conf** file, ensure you add the property **server_name** and assign the value *localhost*: `server_name localhost`. This is required for IDEs, such as PhpStorm, to properly recognize the incomming debug connection. You can reference the included **nginx.conf** file of this repo.

## Step 3: Configuring Xdebug
To finalize your Xdebug setup, you have to configure it in the INI file. These cofnigurations can be made directly in your **php.ini** file or a separate **xdebug.ini** file. For my setup, I have a **php-ini-overrides.ini** file where I can add or override default values.

1. `zend_extensions=xdebug`: This tells PHP that you're using the Xdebug engine. If you enabled this via a command in your **Dockerfile**, then you can exclude or comment this line. Try adding it first, and remove it only if you get an error that it's already enabled.
2. `xdebug.mode=develop,debug`: Xdebug has several modes, which have to be manually enabled. In this setup, the **develop** mode provides better looking and well formatted exceptions and var_dump output. The **debug** mode is what allows for the advanced debugging, stepping through your code and evaluating its state.
3. `xdebug.start_with_request=yes`: Enables the trigger for step debugging. [Read More](https://xdebug.org/docs/step_debug#activate_debugger)
4. `xdebug.discover_client_host=false`: Disables Xdebug from attempting to connect to other clients that might be discovered by other means. [Read More](https://xdebug.org/docs/step_debug#discover_client_host)
5. `xdebug.client_host=host.docker.internal`: Configures the IP address or hostname where Xdebug will attempt to connect to when initiating a debugging connection. [Read More](https://xdebug.org/docs/step_debug#client_host)
