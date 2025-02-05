# Fixing Access Issues: Open Ports on Oracle Cloud's Ubuntu Instance

When deploying a web application on an Ubuntu instance in Oracle Cloud, you may encounter access issues due to restricted firewall settings. If your application is not accessible over the internet, you likely need to open the required ports both in Oracle Cloud's security rules and on your Ubuntu instance.

This guide will walk you through the necessary steps to ensure your web app is accessible.

## Step 1: Verify Your Virtual Cloud Network (VCN)

Before modifying firewall rules, ensure you are working within the correct Virtual Cloud Network (VCN):

1. Navigate to your **Oracle Cloud Console**.
2. Go to **Instance Details**.
3. Click on your **Virtual Cloud Network (VCN)** to confirm you are in the correct network.

---

## Step 2: Modify Security List Rules

To allow traffic to your instance, you need to update the Security List:

1. In the **Oracle Cloud Console**, navigate to **Networking > Virtual Cloud Networks**.
2. Click on your **VCN**, then go to **Security Lists**.
3. Select the **Default Security List** (or the one associated with your instance).
4. Click **Add Ingress Rules** and configure as follows:
   - **Source CIDR**: `0.0.0.0/0` (allows public access; to restrict, provide a specific IP).
   - **Destination Port Range**: `80, 443` (or any other ports your server is listening on).
   - **Protocol**: `TCP`.
5. Save the changes.

At this point, your instance is configured to allow incoming traffic, but your web application might still be inaccessible. This is because Ubuntu runs **iptables**, which needs further configuration.

---

## Step 3: Configure iptables on Ubuntu

Oracle Cloud's Ubuntu instances use **iptables** as their firewall, which may be blocking your ports. To allow incoming traffic, follow these steps:

1. **Connect to your instance via SSH**.
2. Open the iptables rules file for editing:
   ```sh
   sudo nano /etc/iptables/rules.v4
   ```
3. Locate the existing `-A INPUT` rules and add the following lines between them:
   ```sh
   -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
   -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
   ```
4. Save the file and exit (**CTRL + X, then Y, then Enter**).
5. Apply the updated iptables rules:
   ```sh
   sudo iptables-restore < /etc/iptables/rules.v4
   ```

---

## Step 4: Verify the Changes

To confirm the ports are open and accessible:

- **Check iptables rules**:
  ```sh
  sudo iptables -L
  ```
- **Verify your server is listening on the expected ports**:
  ```sh
  sudo netstat -tulnp | grep LISTEN
  ```
- **Test access from your browser or using curl**:
  ```sh
  curl -I http://your-server-ip
  ```

---

## Conclusion

By following these steps, you have successfully configured **Oracle Cloud's security rules** and **Ubuntu's firewall** to allow access to your web application. If you are still experiencing issues, ensure your web server (**Apache, Nginx, etc.**) is running and properly configured to listen on the correct ports.

Now, your website should be accessible to users over the internet!
