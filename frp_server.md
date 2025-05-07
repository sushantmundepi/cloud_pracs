# **FRP (Fast Reverse Proxy) server** 

---

## **1. Prerequisites**

- An **EC2 instance** (named **FRP Instance**) running **Ubuntu**.
- **FRP (Fast Reverse Proxy)** binaries, downloadable from [GitHub Releases](https://github.com/fatedier/frp/releases).
- Open security group ports (e.g., port 7000 for FRP connections).
- Familiarity with **Linux** command-line operations.

--- 

## **Step 1: Launch an EC2 Instance (FRP Instance)**

1. **Sign in to AWS Management Console**.
2. Navigate to **EC2** and select **Launch Instance**.
3. Choose an **Ubuntu AMI** of your preferred version.
4. Pick an instance type (e.g., `t2.micro` for free-tier usage).
5. Configure additional instance settings based on your needs.
6. Add a key pair to securely connect via SSH.
7. Assign a security group (you can create a new one or use an existing one).
8. Click **Launch Instance** and wait for it to be ready.

---

## **Step 2: Modify Security Group to Open Port 7000**

To manually update the security group settings:

1. Open the **EC2 Dashboard**, then go to **Security Groups** (under **Network & Security**).
2. Locate the security group assigned to your **FRP Instance**.
3. Click on the **Inbound rules** tab, then select **Edit inbound rules**.
4. Add a new entry:
   - **Type**: Custom TCP Rule
   - **Port Range**: 7000
   - **Source**: Anywhere (`0.0.0.0/0`) or restrict it to specific IPs.
5. Save the changes by clicking **Save rules**.

Next, establish an SSH connection to your EC2 instance.

---

## **Step 3: Connecting to EC2 via PuTTY**

1. **Install PuTTY**:\
   If not installed, download PuTTY from [this link](https://www.putty.org/) and install it.

2. **Convert PEM Key to PPK**:\
   Since PuTTY doesn’t support `.pem` files directly, convert your key to `.ppk`:

   - Open PuTTYgen
   - Load your `.pem` file
   - Save private key as `.ppk`

3. **Retrieve the Public IP Address** of your EC2 instance from the **EC2 Dashboard** under **Instances**.

4. **Launch PuTTY** and enter:

   - **Host Name (or IP address)**: *Your EC2 Public IP*
   - **Connection Type**: SSH
   - **Authentication**: Load the `.ppk` file

5. **Initiate SSH Connection**:\
   Click **Open**, then log in as ``.

---

## **2. Installing the FRP Server on EC2**

### **Step 1: Install Required Dependencies**

```bash
sudo apt update
sudo apt install wget curl -y
```

### **Step 2: Download and Extract FRP**

```bash
wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
tar -zxvf frp_0.47.0_linux_amd64.tar.gz
cd frp_0.47.0_linux_amd64
```

---

### **Step 3: Move FRP Files to an Organized Directory**

```bash
sudo mkdir -p /opt/frp
sudo mv frps frps.ini /opt/frp/
```

---

### **Step 4: Assign Execution Permissions**

```bash
sudo chmod +x /opt/frp/frps
```

---

## **3. Configuring the FRP Server**

```bash
sudo nano /opt/frp/frps.ini
```

Edit the file as needed:

```ini
[common]
bind_port = 7000
vhost_http_port = 80
vhost_https_port = 443
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin
```

Save (`CTRL + X`, `Y`, `ENTER`).

---

## **4. Running the FRP Server**

```bash
sudo /opt/frp/frps -c /opt/frp/frps.ini
```
![Screenshot (34)](https://github.com/user-attachments/assets/755b7400-8d2f-44ae-90f9-77e52f99acdb)


---

## **5. Setting Up FRP as a Systemd Service**

### **Step 1: Create a systemd Service File**

```bash
sudo nano /etc/systemd/system/frps.service
```

Add the following content:

```ini
[Unit]
Description=FRP server
After=network.target

[Service]
ExecStart=/opt/frp/frps -c /opt/frp/frps.ini
Restart=always
User=nobody
RestartSec=3

[Install]
WantedBy=multi-user.target
```

---

### **Step 2: Reload and Start FRP Service**

```bash
sudo systemctl daemon-reload
sudo systemctl start frps
sudo systemctl enable frps
```

---

### **Step 3: Verify FRP Service Status**

```bash
sudo systemctl status frps
```

It should display ``.

---

## **6. Enabling the FRP Dashboard (Optional)**

If you enabled the dashboard (`dashboard_port = 7500`), access it via:

```
http://<your-ec2-public-ip>:7500
```

Login:

- **Username**: `admin`
- **Password**: `admin`

---

## **7. Connecting the FRP Client**

### **Step 1: Download and Configure the Client**

```bash
wget https://github.com/fatedier/frp/releases/download/v0.47.0/frp_0.47.0_linux_amd64.tar.gz
tar -zxvf frp_0.47.0_linux_amd64.tar.gz
```

Edit `frpc.ini`:

```ini
[common]
server_addr = <your-ec2-public-ip>
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

### **Step 2: Start the Client**

```bash
sudo ./frpc -c ./frpc.ini
```
![Screenshot (37)](https://github.com/user-attachments/assets/20e50db0-4059-4bd5-8ead-74e4cc37e020)


### **Step 3: SSH via FRP Tunnel**

```bash
ssh -p 6000 user@<your-ec2-public-ip>
```

---

## **8. Conclusion**

Your FRP server is successfully deployed on EC2! 🚀 You can now use it for secure port forwarding and remote access.

For advanced configurations, visit the [official FRP documentation](https://github.com/fatedier/frp/blob/master/README.md)
