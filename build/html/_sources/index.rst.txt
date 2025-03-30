READ THE DOCS WITH DOCKER  
=========================  

This is the documentation to build and run the Docker container that hosts Read the Docs, allowing you to edit your documentation anywhere.  
There are two types of repositories: public and private. First, we will set it up with a public repository.  

For Public Repositories
-----------------------

Dockerfile
~~~~~~~~~~
First, you must copy the Dockerfile:  

.. code-block:: docker  

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
   RUN pip install --upgrade pip \  
      && pip install sphinx sphinx_rtd_theme  

   # Configure OpenSSH  
   RUN mkdir /var/run/sshd  
   RUN echo 'root:password' | chpasswd  # Change the password for security  

   # Allow login via SSH  
   RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config  
   RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config  

   # Expose port 22 (you can change the port)  
   EXPOSE 22  

   # Set the working directory  
   WORKDIR /app  

   # Clone your Read the Docs repository (HTTPS)  
   RUN git clone <https://github.com/yourrepo> .  

   # Install `requirements.txt` dependencies if they exist  
   RUN if [ -f "requirements.txt" ]; then pip install -r requirements.txt; fi  

   # Create volumes  
   VOLUME [ "/root/.ssh", "/app" ]  

   CMD ["/usr/sbin/sshd", "-D"]  

Docker set up
~~~~~~~~~~~~~
Once you edit the port (if needed) and your repository URL, run the following command:  

.. code-block:: shell  

   docker build --no-cache -t rtdocker .  

The flag `--no-cache` ensures that Docker does not override the working directory or delete the cloned repository.  

When the build is finished, it's time to run the container. Use this command:  

.. code-block:: shell  

   # Change `hostport` to any available port on your machine  
   # Change `22` if you have modified it in the Dockerfile  
   docker run -d -p hostport:22 --name rtd-container rtdocker  

Now that the container is running, establish a connection with it using:  

.. code-block:: shell  

   # Change `hostport` to the previously set value  
   ssh root@localhost -p hostport  

Type **yes** when prompted, then enter the password you set in the Dockerfile.  

Once inside the container, verify your current directory:  

.. code-block:: shell  

   pwd  

If you are in:  

.. code-block:: shell  

   /root  

Navigate to the application directory:  

.. code-block:: shell  

   cd ..  
   cd app/  

If you are in a different directory, search for the `app` directory, where the cloned repository is located.  

Now you can edit your files and push them to GitHub. Before pushing, make sure you are logged into GitHub.  

Logging into GitHub
~~~~~~~~~~~~~~~~~~~
You can log in however you want; here we are going to show the SSH method:  

For private repositories:
-------------------------

.. toctree::  
   :maxdepth: 2  
   :caption: Contents:  

   index.rst