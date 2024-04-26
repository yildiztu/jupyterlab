# To run JupyterLab from Docker in Ubuntu 22.04, follow the steps below:

1. Install Docker if you haven't done it yet. Run the following commands in your terminal:
   
   ```
   sudo apt update
   sudo apt install docker.io
   ```
   After installation, start the Docker service:
   ```
   sudo systemctl start docker
   ```

2. Pull the official Jupyter Docker image by running the following command:
   
   ```
   sudo docker pull jupyter/datascience-notebook
   ```

3. Run the JupyterLab container using the pulled image by executing the command below. Replace `/path/to/notebook/folder` with the directory path on your Ubuntu system where you want to load the notebooks from:

   ```
   sudo docker run --name jupyterlab -p 8888:8888 -v /home/ubuntu/jupyterlab:/home/jovyan/work jupyter/datascience-notebook
   ```

   See the token value using
   ```
   sudo docker logs jupyterlab
   ```

   or without the need for token value
   ```
   sudo docker run --detach --name jupyterlab -p 8888:8888 -v /home/ubuntu/jupyterlab:/home/jovyan/work jupyter/datascience-notebook start.sh jupyter lab --NotebookApp.token=''
   ```

5. After running the above command, you will see some logs in the terminal. Near the end, you will see a URL starting with `http://127.0.0.1:8888/?token=`, copy that URL.

6. Open a web browser and paste the URL from the previous step. This will open up JupyterLab in your browser.

Now you can start using JupyterLab from Docker in your Ubuntu 22.04 environment. Note that the data in the `/path/to/notebook/folder` directory on your host machine will be accessible within the JupyterLab environment.

Remember to stop and remove the container when you're done with JupyterLab using the following commands:

```
sudo docker stop jupyterlab
sudo docker rm jupyterlab
```

This will clean up the container and remove it from your system.

# To obtain a Let's Encrypt SSL certificate for your JupyterLab instance running on `jupyterlab.mydomain.com`, you can use Certbot, a widely used tool for managing SSL certificates. Here's how you can do it:

1. Install Certbot and the appropriate Certbot plugin for the web server in your Ubuntu 22.04 system by running the following commands:
   
   ```
   sudo apt update
   sudo apt install certbot python3-certbot-nginx
   ```

2. Ensure that your `jupyterlab.mydomain.com` domain is correctly pointed to your Ubuntu server's IP address.

3. Configure a reverse proxy for JupyterLab using Nginx. Create a new configuration file for your domain by running:
   
   ```
   sudo nano /etc/nginx/sites-available/jupyterlab.mydomain.com
   ```

   Add the following configuration to the file, replacing `jupyterlab.mydomain.com` with your actual domain and `8888` with the JupyterLab's port if you have modified it:

   ```
   server {
       listen 80;
       server_name jupyterlab.mydomain.com;

       location / {
           proxy_pass http://localhost:8888;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

   Save and close the file (Ctrl+O, Ctrl+X).

4. Create a symbolic link from the configuration file in the `sites-available` directory to the `sites-enabled` directory by running:
   
   ```
   sudo ln -s /etc/nginx/sites-available/jupyterlab.mydomain.com /etc/nginx/sites-enabled/
   ```

5. Test the Nginx configuration for syntax errors by running:
   
   ```
   sudo nginx -t
   ```
   
   If the command outputs "syntax is ok" without any errors, proceed to the next step. Otherwise, fix any errors in the Nginx configuration file.

6. Restart Nginx to apply the changes:
   
   ```
   sudo systemctl restart nginx
   ```

7. Run Certbot to obtain an SSL certificate by executing the following command:
   
   ```
   sudo certbot --nginx -d jupyterlab.mydomain.com
   ```

   Certbot will automatically configure Nginx to use the obtained SSL certificate and redirect HTTP traffic to HTTPS.

8. Once the certificate is successfully obtained and configured, you can access your JupyterLab instance securely via `https://jupyterlab.mydomain.com`.

Remember to renew the SSL certificate before it expires by using the `certbot renew` command. Let's Encrypt certificates are typically valid for 90 days and can be auto-renewed using a cron job or systemd timer.


# To set a password for JupyterLab, you can use the `jupyter notebook password` command, which is available in the Jupyter Notebook package.

Here are the steps to set a password for JupyterLab:

1. Access the container running JupyterLab using the following command:

```bash
sudo docker exec -it jupyterlab bash
```

Replace `jupyterlab` with the name or ID of your running container.

2. Once inside the container's command prompt, run the following command to create a password:

```bash
jupyter notebook password
```

3. You will be prompted to enter a password. After entering your desired password, press Enter.

4. The command will generate a hashed password. It will look something like this:

```
Enter password:
Verify password:
[NotebookPasswordApp] Wrote hashed password to /home/jovyan/.jupyter/jupyter_notebook_config.json
```

The hashed password is stored in the `.jupyter/jupyter_notebook_config.json` file.

5. Exit the container by typing `exit` in the command prompt.

After setting the password, you can restart the JupyterLab container with the `-v` flag mapping the location of the `.jupyter` directory, which contains the hashed password. This allows the password to persist across container restarts.

Here's an example of how to restart the container with the password configured:

```bash
sudo docker run --detach --name jupyterlab -p 8888:8888 -v /home/ubuntu/jupyterlab:/home/jovyan/work -v /home/ubuntu/jupyterlab/config:/home/jovyan/.jupyter jupyter/datascience-notebook
```

In the above command, `/home/ubuntu/jupyterlab/config` is the local directory where you want to store the JupyterLab configuration files, including the hashed password.

After restarting the container, the password you set will be required to access JupyterLab.

