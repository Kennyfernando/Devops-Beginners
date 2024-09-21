# Setting up a Load Balancer for Static Website Using Nginx

Please reference [**Project1**](https://github.com/Kennyfernando/Devops-Beginners/blob/main/project1/project1.md) for guidance on spinning up an Ubuntu server, as well as creating and associating an elastic IP address with your server, among other tasks.

- spin 3 ubuntu instance and name them accordingly and clearly

1 **server-1**

2 **server-2**

3 **load-balancer**

![1](img/1.png)

### Install Nginx and setup your website.

- Download your prefered website template from `tooplate.com`

- choose your template and right click, then select **inspect** from the drop down menu.

![2](img/2.png)

- Click on the **Network** tab.

![3](img/3.png)

- Click the **Download** button and **right click on the website name**.

- To install Nginx, execute the following commands on your terminal.

`sudo apt update`

`sudo apt upgrade`

`sudo apt install nginx`

- Start your Nginx server by running the `sudo systemctl start nginx` command, enable it to start on boot by executing `sudo systemctl enable nginx`, and then confirm if it's running with the `sudo systemctl status nginx` command.

![5](img/4.png)

> [!NOTE]
Install Nginx on both web server terminals. These are the terminals you're using to manage the servers hosting the two distinct website contents that the load balancer will distribute traffic to.

- Visit your instances IP address in a web browser to view the default Nginx startup page.

### Setting Up Your Website.

- Execute `sudo apt install unzip` to install the unzip tool and run the following command to download and unzip your website files `sudo curl -o /var/www/html/2132_clean_work.zip https://www.tooplate.com/zip-templates/2132_clean_work.zip && sudo unzip -d /var/www/html/ /var/www/html/2132_clean_work.zip && sudo rm -f /var/www/html/2132_clean_work.zip`.

![6](img/5.png)

- Creat a file in the Nginx sites-availaible with the following command `sudo nano /etc/nginx/sites-available/clean`

- Copy and paste the following code intothe open text editor

```
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/html/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- Edit the `root` directive within your server block to point to the directory where your downloaded website content is stored.

![7](img/6.png)

- Run the command below to create a symbolic link for both websites
`sudo ln -s /etc/nginx/sites-available/clean /etc/nginx/sites-enabled/`

-  Run the `sudo nginx -t` command to check the syntax of the Nginx configuration file, and when successful run the `sudo systemctl restart nginx` command.


![8](img/7.png)

- Run the `sudo rm -f /etc/nginx/sites-enabled/default` to delete the default sites-enabled folders and enable nginx to serve content from your specified website directories. If you dont delete the folder you will continue to see the th default Nginx page.

![9](img/10.png)

- Repeat the process for the second website.

> [!NOTE]
To better identify the impact of your changes, connect to the second server in a new terminal window. There, update the website content with something different. When you reload your website, the load balancer will distribute traffic, potentially sending you to the updated version on the second server. This will make the differences between the two versions clearer.

- Execute `sudo apt install unzip` to install the unzip tool and run the following command to download and unzip your website files `sudo curl -o /var/www/html/2135_mini_finance.zip https://www.tooplate.com/zip-templates/2135_mini_finance.zip && sudo unzip -d /var/www/html/ /var/www/html/2135_mini_finance.zip && sudo rm -f /var/www/html/2135_mini_finance.zip`.

![10](img/11.png)

- To set up your website's configuration, start by creating a new file in the Nginx sites-available directory. Use the following command to open a blank file in a text editor: `sudo nano /etc/nginx/sites-available/finance`.

- Copy and paste the following code into the open text editor.

```
server {
    listen 80;
    server_name placeholder.com www.placeholder.com;

    root /var/www/html/placeholder.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

- Edit the `root` directive within your server block to point to the directory where your downloaded website content is stored.

![11](img/12.png)

- Create a symbolic link for both websites by running the following command.
`sudo ln -s /etc/nginx/sites-available/finance /etc/nginx/sites-enabled/`

- Run the `sudo nginx -t` command to check the syntax of the Nginx configuration file, then run `sudo rm /etc/nginx/sites-enabled/default.

![alt text](img/13.png)

- And when successful run the `sudo systemctl restart nginx` command.

![alt text](img/14.png)

- Check both IP addresses to confirm your website is up and running.

![clean](img/15.png)

![finance](img/16.png)

---

   ### Configure your load-balancer

   - Install Nginx on the server you want to use as a load balancer, and execute `sudo systemctl status nginx` to ensure it's running.

![17](img/17.png)

- Execute `sudo nano /etc/nginx/nginx.conf` to edit your Nginx configuration file.

- Add the following within the http block.

```

    upstream nandez-tech {
    server 1;
    server 2;
    # Add more servers as needed
}

server {
    listen 80;
    server_name <your domain> www.<your domain>;

    location / {
        proxy_pass http://nandez-tech;
    }
}

```

![alt text](img/24.png)

> [!NOTE]
Replace the necessary placeholders as shown in the picture above. Substitute `<server 1>` and `<server 2>` with the actual private IP addresses of your servers. Also, replace `<your domain> www.<your domain>` with your root domain and subdomain name, and update proxy_pass and the other relevant fields accordingly.

- Run `sudo nginx -t` to check for syntax error.

![alt text](img/19.png)

- Apply the changes by restarting Nginx:
`sudo systemctl restart nginx`

---

### Create An A Record

To make your website accessible via your domain name rather than the IP address, you'll need to set up a DNS record. I did this by buying my domain from Namecheap and then moving hosting to AWS Route 53, where I set up an A record.

> [!NOTE]
Please reference [**Project1**](https://github.com/Kennyfernando/Devops-Beginners/blob/main/project1/project1.md) for instructions on how to create a hosted zone.

- Point your domain's A records to the IP address of your Nginx load balancer server.

- In route 53, select the domain name and click on **Create record**.

- Paste your **IP address➀** and then click on **Create records➁** to create the root domain.

!![alt text](img/20.png)

- Click on **create record** again, to create the record for your sub domain.

- Paste your IP address➀, input the Record name(**www➁**) and then click on **Create records**➂.

![21](img/21.png)

- Go to the terminal you used in setting your first website and run `sudo nano /etc/nginx/sites-available/finance` to edit your settings. Enter the name of your domain and then save your settings.

![22](img/22.png)

- Restart your nginx server by running the `sudo systemctl restart nginx` command.

- Go to the terminal you used in setting your second website and run `sudo nano /etc/nginx/sites-available/interior` to edit your settings. Enter the name of your domain and then save your settings.

![24](img/23.png)

- Restart your nginx server by running the `sudo systemctl restart nginx` command.

- Go to your domain name in a web browser to verify that your website is accessible.

![26](img/26.png)

- Reload the webpage to ensure the load balancer distributes traffic evenly between your servers.

![alt text](img/25.png)


---

### Install certbot and Request For an SSL/TLS Certificate

- Install certbot by executing the following commands:
`sudo apt update`
`sudo apt install python3-certbot-nginx`

- Execute the `sudo certbot --nginx` command to request your certificate. Follow the instructions provided by certbot and select the domain name for which you would like to activate HTTPS.

![28](img/28.png)

- You should get a congratulatory message that says https has been successfully enabled.

![29](img/29.png)

- Access your website to verify that Certbot has successfully enabled HTTPS.

- It is recommended to renew your LetsEncrypt certificate at least every 60 days or more frequently. You can test renewal command in dry-run mode:
`sudo certbot renew --dry-run`

![30](img/30.png)

![alt text](img/31.png)

---
---

#### The End Of Project 3