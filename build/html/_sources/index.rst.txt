```rst
==========================
READ THE DOCS WITH DOCKER
==========================

This is the documentation to build and run the Docker container that hosts Read the Docs, allowing you to edit your documentation anywhere.  
There are two types of repositories: public and private. First, we will set it up with a public repository.  

For Public Repositories
=======================

.. toctree::
   :maxdepth: 1

   dockerfile
   docker_setup
   logging_into_github

Dockerfile
==========

First, you must copy the Dockerfile:  

.. code-block:: dockerfile

   FROM ubuntu:latest  

   USER root  

   # Install system dependencies  
   RUN apt-get update && apt-get install -y \  
       git \  
       openssh-server \  
       python3 \  
       python3-pip \  
       python3-venv \  
       python3-full \  
       vim    

   # Create a Python virtual environment  
   RUN python3 -m venv /venv  
   ENV PATH="/venv/bin:$PATH"  

   # Install Sphinx and sphinx_rtd_theme  
   RUN pip install --upgrade pip && pip install sphinx sphinx_rtd_theme  

   # Configure OpenSSH  
   RUN mkdir /var/run/sshd  
   RUN echo 'root:password' | chpasswd  

   # Allow login via SSH  
   RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config  
   RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config  

   # Expose port 22  
   EXPOSE 22  

   # Set the working directory  
   WORKDIR /app  

   # Clone your Read the Docs repository  
   RUN git clone <https://github.com/yourrepo> .  

   # Install `requirements.txt` dependencies if they exist  
   RUN if [ -f "requirements.txt" ]; then pip install -r requirements.txt; fi  

   # Create volumes  
   VOLUME [ "/root/.ssh", "/app" ]  

   CMD ["/usr/sbin/sshd", "-D"]  

Docker Setup
============

Once you edit the port (if needed) and your repository URL, run the following command:  

.. code-block:: shell

   docker build --no-cache -t rtdocker .  

When the build is finished, run the container:  

.. code-block:: shell

   docker run -d -p hostport:22 --name rtd-container rtdocker  

Now that the container is running, establish a connection with it using:  

.. code-block:: shell

   ssh root@localhost -p hostport  

Once inside the container, verify your current directory:  

.. code-block:: shell

   pwd  

If you are in:  

.. code-block:: shell

   /root  

Navigate to the application directory:  

.. code-block:: shell

   cd /app  

Now you can edit your files and push them to GitHub.  

Logging into GitHub
===================

To set up GitHub via SSH, write the following commands:  

.. code-block:: shell

   ssh-keygen -t ed25519 -C "youremail@example.com"  
   eval "$(ssh-agent -s)"  
   ssh-add ~/.ssh/id_ed25519  
   cat ~/.ssh/id_ed25519.pub  

Now, copy the output and go to GitHub:  

1. Click on **Settings** → **SSH and GPG keys** → **New SSH Key**.  
2. Paste the copied key and click **Add**.  

Go back to Docker and verify the connection:  

.. code-block:: shell

   ssh -T git@github.com  

You should see:  

.. code-block:: shell

   Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.  

Pushing to GitHub
=================

Before starting, make sure you are in the folder where the repository is located.  

.. code-block:: shell

   git remote set-url origin git@github.com:user/repository.git  
   git add .  
   git commit -m "Your message"  
   git push origin main  

For Private Repositories
========================

.. toctree::
   :maxdepth: 2
   :caption: READ THE DOCS WITH DOCKER

   public_repositories/index
   private_repositories/index
```