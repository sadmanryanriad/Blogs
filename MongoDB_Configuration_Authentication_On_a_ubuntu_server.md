# MongoDB Installation and Configuration on Ubuntu VPS

MongoDB is a popular NoSQL database that can be easily installed and configured on an Ubuntu VPS. In this article, we'll walk through the process of installing MongoDB on an Ubuntu server, setting up users with authentication and authorization, and securely accessing the database from your local machine using SSH port forwarding and MongoDB Compass.

## Table of Contents

- [How to Install MongoDB on Ubuntu VPS](#how-to-install-mongodb-on-ubuntu-vps)
- [Creating Users with Specific Roles](#creating-users-with-specific-roles)
- [Enabling Authentication and Authorization](#enabling-authentication-and-authorization)
- [Configuring MongoDB to Bind to IP Addresses](#configuring-mongodb-to-bind-to-ip-addresses)
- [Using SSH Port Forwarding for Secure Remote Access](#using-ssh-port-forwarding-for-secure-remote-access)
- [Updating Admin Permissions for Full Database Access](#updating-admin-permissions-for-full-database-access)
- [Connecting to MongoDB from Your Local Machine Using MongoDB Compass](#connecting-to-mongodb-from-your-local-machine-using-mongodb-compass)
- [Conclusion](#conclusion)

## 1. How to Install MongoDB on Ubuntu VPS <a name="how-to-install-mongodb-on-ubuntu-vps"></a>

### Step 1: Update Your System

Before installing MongoDB, it’s always a good practice to update your system packages:

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Import the MongoDB GPG Key

Import the MongoDB public GPG key to ensure secure package installation:

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
```

### Step 3: Add MongoDB Repository

Next, add the MongoDB repository to your sources list:

```bash
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```

### Step 4: Install MongoDB

Now, install MongoDB using the following commands:

```bash
sudo apt update
sudo apt install -y mongodb-org
```

### Step 5: Start and Enable MongoDB

Start the MongoDB service and enable it to start automatically on system boot:

```bash
sudo systemctl start mongod
sudo systemctl enable mongod
```

### Step 6: Verify the Installation

Check the status of MongoDB to ensure it is running:

```bash
sudo systemctl status mongod
```

If MongoDB is running, you're all set!

## 2. Creating Users with Specific Roles <a name="creating-users-with-specific-roles"></a>

Before enabling security, it’s crucial to create an admin user for your MongoDB instance.

### Step 1: Log into the MongoDB Shell

Access the MongoDB shell as the default system user:

```bash
mongosh
```

### Step 2: Create an Admin User

First, switch to the admin database:

```javascript
use admin
```

Create a new admin user with appropriate roles:

```javascript
db.createUser({
  user: "adminUser",
  pwd: "strongPassword",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" },
  ],
});
```

This admin user can now manage users and databases across the server.

### Step 3: Create a Database-Specific User

Next, create a new user for a specific database (e.g., `testdb`):

```javascript
use testdb
db.createUser({
  user: "testUser",
  pwd: "securePassword",
  roles: [ { role: "readWrite", db: "testdb" } ]
})
```

This user will have readWrite access to the `testdb` database.

## 3. Enabling Authentication and Authorization <a name="enabling-authentication-and-authorization"></a>

By default, MongoDB does not enforce user authentication. Now that you've created users, you can enable authentication and authorization to secure your MongoDB instance.

### Step 1: Edit the MongoDB Configuration File

Open the MongoDB configuration file (`/etc/mongod.conf`) using a text editor:

```bash
sudo nano /etc/mongod.conf
```

### Step 2: Enable Authorization

Find the security section in the configuration file and add the following lines to enable authorization:

```yaml
security:
  authorization: enabled
```

Save the file and exit the editor.

### Step 3: Restart MongoDB

Restart MongoDB for the changes to take effect:

```bash
sudo systemctl restart mongod
```

## 4. Configuring MongoDB to Bind to IP Addresses <a name="configuring-mongodb-to-bind-to-ip-addresses"></a>

By default, MongoDB only binds to `localhost` (`127.0.0.1`). If you need to allow remote access, you’ll need to update the `bindIp` configuration.

### Step 1: Edit the MongoDB Configuration File

Open the `mongod.conf` file:

```bash
sudo nano /etc/mongod.conf
```

### Step 2: Modify the bindIp Setting

Locate the `net` section and modify the `bindIp` to allow remote access. You can bind it to a specific IP or to all interfaces:

```yaml
net:
  bindIp: 127.0.0.1,<your_public_ip>
```

Or, to allow connections from any IP (less secure, for testing only):

```yaml
net:
  bindIp: 0.0.0.0
```

Don't forget to change firewall settings.

### Step 3: Restart MongoDB

Restart the MongoDB service:

```bash
sudo systemctl restart mongod
```

## 5. Using SSH Port Forwarding for Secure Remote Access <a name="using-ssh-port-forwarding-for-secure-remote-access"></a>

To securely access MongoDB from your local machine without exposing MongoDB to the internet, you can use SSH port forwarding.

### Step 1: Open a Terminal on Your Local Machine

Run the following command to create an SSH tunnel:

```bash
ssh -L 27018:localhost:27017 <your_vps_username>@<your_vps_ip>
```

This command forwards port 27017 on the VPS to port 27018 on your local machine.

### Step 2: Keep the Tunnel Open

Leave the terminal running to keep the SSH tunnel active. You can now access MongoDB on your local machine through port 27018.

## 6. Updating Admin Permissions for Full Database Access <a name="updating-admin-permissions-for-full-database-access"></a>

After creating the `adminUser`, you may want them to have full access to manage and view all databases. By default, the user creation process grants only user management privileges. To allow the admin user to view, read, and write to all databases, you need to update their roles.

### Step 1: Log into the MongoDB Shell

Access the MongoDB shell in your VPS:

```bash
mongosh
```

### Step 2: Switch to the Admin Database

Use the admin database:

```javascript
use admin
```

### Step 3: Update the Admin User’s Roles

Run the following command to give the `adminUser` full access to all databases:

```javascript
db.updateUser("adminUser", {
  roles: [
    { role: "readWriteAnyDatabase", db: "admin" },
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" },
  ],
});
```

This command assigns three roles to the `adminUser`:

- **readWriteAnyDatabase**: Grants read and write access to all databases.
- **userAdminAnyDatabase**: Allows the user to manage other users across all databases.
- **dbAdminAnyDatabase**: Grants administrative rights (e.g., creating indexes, viewing collections, managing database properties) across all databases.

In short, these roles allow the admin user to view, manage, and modify all databases from MongoDB Compass or the MongoDB shell using admin.

## 7. Connecting to MongoDB from Your Local Machine Using MongoDB Compass <a name="connecting-to-mongodb-from-your-local-machine-using-mongodb-compass"></a>

Once you have SSH port forwarding set up, you can connect to your MongoDB server using MongoDB Compass.

### Step 1: Open MongoDB Compass

Open MongoDB Compass on your local machine.

### Step 2: Enter the Connection String

In the connection string field, enter the following:

```bash
mongodb://<username>:<password>@localhost:27018/testdb?authSource=admin
```

Replace `<username>`, `<password>`, and `testdb` with your user’s credentials and database.

### Step 3: Test the Connection

Click **Connect**. MongoDB Compass will connect to the MongoDB instance on your VPS through the SSH tunnel.

## Conclusion <a name="conclusion"></a>

By following this guide, you’ve successfully installed MongoDB on your Ubuntu VPS, enabled authentication and authorization, and set up SSH port forwarding for secure remote access. You can now manage your databases securely using MongoDB Compass and ensure your MongoDB instance is protected from unauthorized access.

This setup ensures you can securely manage your MongoDB instance, whether for development or production, without exposing it
