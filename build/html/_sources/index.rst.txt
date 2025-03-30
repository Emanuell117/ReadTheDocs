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
   RUN git clone https://github.com/yourrepo .  

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

   cd /app  

If you are in a different directory, search for the `app` directory, where the cloned repository is located.  

Now you can edit your files and push them to GitHub. Before pushing, make sure you are logged into GitHub.  

Logging into GitHub
~~~~~~~~~~~~~~~~~~~
You can log in however you want; here we are going to show the SSH method.

To set up your GitHub via SSH, write the following commands:

.. code-block:: shell  

   # Change your email  
   ssh-keygen -t ed25519 -C "youremail@example.com"  
   eval "$(ssh-agent -s)"  
   ssh-add ~/.ssh/id_ed25519  
   cat ~/.ssh/id_ed25519.pub  

Now copy the output of `cat`, then go to GitHub.  
Click on your profile → Settings → SSH and GPG keys → New SSH Key.  
In the key field, paste the output, then click "Add".  

Go back to Docker and write:  

.. code-block:: shell  

   ssh -T git@github.com  
   # You should see an output like this:  
   Hi your-username! You've successfully authenticated, but GitHub does not provide shell access.  

Pushing to GitHub
~~~~~~~~~~~~~~~~~
Remember, to push to the repository, you must have an account with access and privileges.

Before we start, make sure you are in the folder where the repository is located.  
Go to GitHub, click on your repository, then click the green "Code" button and copy the SSH URL.  
Follow these steps:  

.. code-block:: shell  

   # Change git@github.com:user/repository.git for the SSH URL  
   git remote set-url origin git@github.com:user/repository.git  

   # Add changes  
   git add .  

   # Make a commit  
   git commit -m "Your message"  

   # Push to GitHub  
   git push origin main  

For Private Repositories
-------------------------

.. toctree::
   :maxdepth: 2
   :caption: READ THE DOCS WITH DOCKER
