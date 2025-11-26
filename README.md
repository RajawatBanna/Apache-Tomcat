# 🚀 Apache Tomcat Setup Guide (Linux)

A complete reference for installing, configuring, securing, and running Apache Tomcat as a Linux service.

---

## 📌 Useful Resources

| Purpose | Link |
|--------|------|
| Apache Tomcat | https://tomcat.apache.org/ |
| OpenLogic Java (OpenJDK) | https://www.openlogic.com/openjdk-downloads |
| Sample WAR File | https://tomcat.apache.org/tomcat-9.0-doc/appdev/sample/ |

---

# ⚙️ 1. Configure `JAVA_HOME` for Tomcat

Create or edit:

```
/opt/tomcat/bin/setenv.sh
```

Add:

```bash
export JAVA_HOME="/usr/lib/jvm/java-openjdk"
```

Make it executable:

```bash
sudo chmod +x /opt/tomcat/bin/setenv.sh
```

---

# 🔐 2. Tomcat Admin Manager Configuration

To enable the Manager GUI, add this line to:

```
conf/tomcat-users.xml
```

```xml
<user username="admin" password="admin" roles="manager-gui"/>
```

> ⚠️ For production, use strong passwords and role-based users.

---

# 🔒 3. Generate an SSL Certificate

Create a 3-year (1095 days) self-signed SSL certificate:

```bash
keytool -genkey   -keyalg RSA   -keystore ssh.jks   -keysize 2048   -validity 1095   -dname "CN=localhost"   -keypass secret   -storepass changeit
```

This generates `ssh.jks`.

---

# 🌐 4. Configure SSL in `server.xml`

Add the following connector block inside:

```
conf/server.xml
```

```xml
<Connector 
    protocol="org.apache.coyote.http11.Http11NioProtocol"
    port="8443"
    maxThreads="200"
    maxParameterCount="1000"
    scheme="https"
    secure="true"
    SSLEnabled="true"
    keystoreFile="ssh.jks"
    keystorePass="changeit"
    clientAuth="false"
    sslProtocol="TLS"
 />
```

Restart Tomcat after saving.

---

# 🛠️ 5. Create a systemd Service for Tomcat

Create the service file:

```bash
sudo vim /etc/systemd/system/tomcat.service
```
Create tomcat user and transfer ownership to tomcat user

```bash
sudo useradd -g tomcat tomcat
sudo chown -R tomcat:tomcat /opt/tomcat
```

Add:

```ini
[Unit]
Description=Apache Tomcat Application Server
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/opt/java/"
ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

Save & exit.

---

# ▶️ 6. Enable and Start Tomcat with systemd

```bash
sudo systemctl daemon-reload
sudo systemctl start tomcat
sudo systemctl enable tomcat
```

### What each command does:
| Command | Purpose |
|---------|---------|
| `daemon-reload` | Loads the updated service file |
| `start tomcat` | Starts the Tomcat server |
| `enable tomcat` | Auto-starts Tomcat on boot |

---

# 🎉 Tomcat is Now Running as a System Service!
