## DeepRacer For Cloud - Ubuntu 24.04 (noble) with 5.2.2 images and NVIDIA 535 drivers

Just some notes on building and configuring a bare-metal install for DeepRacer.  As always - your mileage may vary

Starting Point:

*   System Name: leota
*   OS: Ubuntu 24.04 noble
*   Kernel: x86\_64 Linux 6.8.0-31-generic
*   Disk: 
*   CPU: 11th Gen Intel Core i7-11700 @ 16x 4.8GHz \[63.0°C\]
*   GPU: Tesla M40 24GB
*   RAM: 31860MiB

### **Install OS**

*   Download ISO from https://ubuntu.com/download/server [this file](https://ubuntu.com/download/server/thank-you?version=24.04&architecture=amd64&lts=true)
*   Make bootable USB with [Balena Etcher](https://github.com/balena-io/etcher)
*   Note:
    *   via the installation process, enable ssh server
    *   DO NOT install the docker (will do that later)
*   run all new system commands

```plaintext
sudo apt update && sudo apt upgrade -y
```

### Docker Install

Make sure that Ubuntu did not snap docker in…

```plaintext
sudo snap remove docker --purge
```

Follow directions from [Docker site](https://docs.docker.com/engine/install/ubuntu/)

1.  [Uninstall old versions](https://docs.docker.com/engine/install/ubuntu/#uninstall-old-versions)
2.  [Install using the `apt` repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
    1.  ```plaintext
        sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        ```
        
3.  [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)
4.  [Configure Docker to start on boot with systemd](https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot-with-systemd)
5.  [Configure default logging driver](https://docs.docker.com/engine/install/linux-postinstall/#configure-default-logging-driver)

Reboot after all the installing is done to make sure environment is fresh and all the services are spun up

```plaintext
docker -v
```

> `Docker version 26.1.3, build b72abbb`

```plaintext
docker-compose -v
```

> `docker-compose version 1.29.2, build unknown`

### Install Python3.11

By default, Noble installs the latest and greatest 3.12x.. but that seems to play havoc with all the DeepRacer scripts - unless you want to dive in to virtual environments (venv).  Since this is a dedicated server that does NOTHING other than DeepRacer, it is an acceptable “downgrade”

1.  Update and Upgrade Open a terminal and execute the following commands to update and upgrade your system packages:
    1.  ```plaintext
        sudo apt update && sudo apt upgrade -y
        ```
        
2.  Install Required Dependencies To compile and install Python 3.11, we need to install the necessary dependencies. Run the following command:
    1.  ```plaintext
        sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev
        ```
        
3.  Download and Extract Python 3.11 Download the Python 3.11 source code using the `wget` command (Tip: do this in a temp / throwaway directory… just need a working/scratch spot to build and install. After it is done, you can delete the directory)
    1.  <table><tbody><tr><td><strong>Download:</strong></td><td><pre><code class="language-plaintext">wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz</code></pre></td></tr><tr><td><strong>Extract the &nbsp;archive:</strong></td><td><pre><code class="language-plaintext">tar -xf Python-3.11.0.tgz</code></pre></td></tr></tbody></table>
        
4.  Navigate / Configure and Build Python 3.11:
    
    <table><tbody><tr><td><strong>Navigate:</strong></td><td><pre><code class="language-plaintext">cd Python-3.11.0</code></pre></td></tr><tr><td><strong>Configure the build with optimization flags:</strong></td><td><pre><code class="language-plaintext">./configure --enable-optimizations</code></pre></td></tr><tr><td><p><strong>Build Python using multiple&nbsp;</strong></p><p><strong>processors to speed up the process:</strong></p></td><td><pre><code class="language-plaintext">make -j$(nproc)</code></pre></td></tr></tbody></table>
    
5.  Install Python Install Python 3.11 on your system:
    1.  ```plaintext
        sudo make altinstall
        ```
        
6.  Verify the Installation Check the Python version to ensure that the installation was successful:
    1.  ```plaintext
        python3.11 --version
        ```
        
    2.  `Python 3.11.0`
7.  Set Python 3.11 as the Default to conveniently access Python 3.11 using the `python3` and pip commands via some aliases. Note: Yes, there are better ways and unlimited ways to do this, but this is HOW I did it… 
    1.  Validate that Python3.11 is installed in the expected location:
        1.  ```plaintext
            which python3.11
            ```
            
        2.  `/usr/local/bin/python3.11`
    2.  edit the `~/.bashrc` and add these aliases at the bottom:
        1.  ```plaintext
            alias python3='/usr/local/bin/python3.11'
            alias python='/usr/local/bin/python3.11'
            alias pip3='/usr/local/bin/pip3.11'
            alias pip='/usr/local/bin/pip3.11'
            ```
            
    3.  Save the file / exit the editor.  Now need to reload your bashrc file.  Just logoff and re-login or 
        1.  ```plaintext
            source ~/.bashrc
            ```
            
    4.  check that the aliases are correct
        1.  ```plaintext
            python3 --version
            python --version
            pip3 --version
            pip --version
            ```
            
8.  Upgrade pip
    1.  ```plaintext
        pip install --upgrade pip
        ```
        
9.  Thats it!  Sure you can take Python 3.11.- to 3.11.9 - but that is left to the reader…

### Getting a PATH

Going forward, there might be come PATH complaining… the default ~./.profile should pick it up once sourced.

```plaintext
source ./profile
```

Look for `/home/<user>/.local/bin` in your path (where \<user> is the login user of your server.  Alternatively, if you want to create a possible problem in the future, but fix the issue _right now_, then edit the with `sudo vim /etc/environment` and add  :`/home/<user>/.local/bin` at the end (keep the closed quotes!
