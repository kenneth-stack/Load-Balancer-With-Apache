# Apache Load Balancer Deployment for Tooling Website Solution

![](/images/Screenshot%20(593).png)

#### Prerequisites

Before starting this project, ensure you have completed the setup described in the [DevOps Tooling Website Solution](https://github.com/kenneth-stack/DevOps-Tooling-website-solution.git). This includes:

- NFS Server
- Database Server
- Two Web Servers

Additionally, you should have:

- Basic understanding of Linux systems and the command line
- Familiarity with Apache web server
- Access to an AWS account

## Project Objectives

1. Implement a load balancing solution using Apache
2. Ensure even distribution of traffic across the three web servers
3. Improve the overall scalability and reliability of the DevOps tooling website
4. Gain practical experience in configuring a load balancer

## Implementation Steps

## Step 1: Launch the Apache Load Balancer

Create an Ubuntu 20.04 EC2 instance and name it Apache Load Balancer and open TCP port 80 by creating an inbound rule in the security group for this instance.
![](/images/Screenshot%20(585).png)

### Step 2: Install Apache and Configure as a Load Balancer

1. Connect to your EC2 instance via SSH:

```bash
   ssh -i <your-private-key.pem> ubuntu@<your-instance-public-ip>
```

2. Update the server:

```bash
   sudo apt update
   sudo apt upgrade -y
```
![](/images/Screenshot%20(586).png)

3. Install Apache and required modules:

```bash
   sudo apt install apache2 -y
   sudo apt install libxml2-dev -y
 ```
 ![](/images/Screenshot%20(587).png)

```bash
   sudo systemctl start apache2
   sudo systemctl enable apache2
```
![](/images/Screenshot%20(588).png)

4. Enable necessary modules:

```bash
   sudo a2enmod rewrite
   sudo a2enmod proxy
   sudo a2enmod proxy_balancer
   sudo a2enmod proxy_http
   sudo a2enmod headers
   sudo a2enmod lbmethod_bytraffic
```
![](/images/Screenshot%20(589).png)

5. Restart Apache to apply changes:

```bash
   sudo systemctl restart apache2
 ```

6. Check Apache status to ensure it's running:
```bash
   sudo systemctl status apache2
```
![](/images/Screenshot%20(590).png)

### Step 3: Configure Load Balancing

1. Open Apache's default configuration file:

```bash
   sudo nano /etc/apache2/sites-available/000-default.conf
```
Add the following configuration within the <VirtualHost *:80> section:

```bash
<VirtualHost *:80>
    <Proxy "balancer://mycluster">
        BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
        BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
        ProxySet lbmethod=bytraffic
        # ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPreserveHost On
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```
> Replace <WebServer1-Private-IP-Address> and <WebServer2-Private-IP-Address> with the private IPs of your two web servers.

![](/images/Screenshot%20(592).png)

>**Load Balancing Methods**
Apache provides several load balancing methods. The method is specified by the lbmethod parameter in the ProxySet directive. Here's a summary of available methods:
> 1. **byrequests (default)**: Distributes requests based on the number of requests that each worker has served. This ensures that each backend server handles an equal number of requests over time.
> 2. **bytraffic**: Distributes requests based on the amount of network traffic each worker has handled. This is useful when requests vary significantly in size.
> 3. **bybusyness**: Distributes requests to the worker with the least active connections. This is useful when backend servers queue requests independently of Apache.
> 4. **heartbeat**: Uses server health checks to distribute requests. This requires additional modules and configuration.
>
>To change the load balancing method, simply update the lbmethod parameter in the ProxySet directive. For example:

```bash
ProxySet lbmethod=bytraffic
```

Choose the method that best suits your application's needs and traffic patterns.

4. Save the file and exit the editor

5. Restart Apache to apply the changes:
```bash
   sudo systemctl restart apache2
```

>*NB: adjust the security group for the web servers to only allow traffic to the port 80 from the load balancer only while the load balancer port 80 is open to the public.*

### Step 4: Verify Load Balancer Configuration

1. Open a web browser and enter your load balancer's public IP address

- Note: If in the previous project, you mounted /var/log/httpd/ from your Web Servers to the NFS server - unmount them and make sure that each Web Server has its own log directory, do the following steps on the two web servers:

    - Check Existing NFS Mounts:
```bash
    df-h
```

- Stop Apache to Prevent Logging Disruption:
```bash
    sudo systemctl stop httpd
```

- To unmount the /var/log/httpd/ directory, run:
```bash
    sudo umount /var/log/httpd
```

- Verify that the unmount was successful by running `df -h`.
> This command should no longer display the /var/log/httpd mount.

- Check /etc/fstab for Persistent NFS Mount:

- Open the /etc/fstab file with a text editor:
```bash
sudo vi /etc/fstab
```
- Find the line that mounts the NFS directory (it will look similar to this):
```bash
    <nfs-server-IP>:/mnt/logs    /var/log/httpd    nfs    defaults    0 0
```

Comment out the line by adding a # at the beginning, or delete the line entirely:
```bash
  <nfs-server-IP>:/mnt/logs    /var/log/httpd    nfs    defaults    0 0
```

- Save and close the file.
- Recreate the Local Log Directory
- If the /var/log/httpd/ directory was mounted from NFS, it may no longer exist locally on the web server. Recreate it:
```bash
        sudo mkdir -p /var/log/httpd
```

- Set the appropriate ownership and permissions:
```bash
    sudo chown apache:apache /var/log/httpd
    sudo chmod 755 /var/log/httpd
```
- Restart Apaches
```bash
    sudo systemctl start httpd
```

> If configured correctly, the requests will be balanced between the two web servers.

2. On both RHEL8 web servers, monitor incoming requests by running:

```bash
sudo tail -f /var/log/httpd/access_log
```
- Refresh the browser page several times, and you should see requests being distributed evenly across both servers

## Conclusion

This project enhances the DevOps Tooling Website Solution by implementing a load balancing solution using Apache. This improvement distributes traffic evenly among the web servers, increasing the scalability and reliability of the application. Through this process, you've gained practical experience in configuring a load balancer and improved your understanding of web application architecture.