# ansible-git-apache-mysql

### Step-by-Step Guide to Setting Up a LAMP Stack with Ansible
![[ansible-deploy-git-apacge-mysql.png]]

#### Step 1: Prepare Your Ansible Environment

1. **Ensure Ansible is Installed**:
    
    - Verify that Ansible is installed on your control node (the machine from which you'll be running Ansible). You can install it using the steps mentioned in the previous response if it's not already installed.

```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```
1. **Set Up SSH Access**:
    
    - Ensure you have SSH access to all the target servers where you will set up the LAMP stack. Passwordless SSH access using SSH keys is recommended for ease of use with Ansible.

#### Step 2: Create Your Inventory File

An inventory file defines the servers where Ansible will run the playbooks.

1. **Create an Inventory File**:
    - Create a file named `hosts` in your project directory with the following content:

	```hosts
	[lamp_servers]
	server1.example.com
	server2.example.com
	```

	- Replace `server1.example.com` and `server2.example.com` with the actual hostnames or IP addresses of your target servers.

#### Step 3: Write the Ansible Playbook

A playbook defines the tasks Ansible will perform. For this project, we will create a playbook to install and configure the LAMP stack components: Apache, MySQL, and PHP.

1. **Create a Playbook File**:
    
    - Create a file named `setup_lamp.yml` in your project directory with the following content:

```yaml
---
- name: Set up LAMP stack on servers
  hosts: lamp_servers
  become: yes

  vars:
    mysql_root_password: "YourSecurePassword"  # Replace with your desired MySQL root password

  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present

    - name: Start and enable Apache
      service:
        name: httpd
        state: started
        enabled: yes

    - name: Install MySQL
      yum:
        name: mariadb-server
        state: present

    - name: Start and enable MySQL
      service:
        name: mariadb
        state: started
        enabled: yes

    - name: Set MySQL root password
      mysql_user:
        name: root
        host: "{{ item }}"
        password: "{{ mysql_root_password }}"
        priv: "*.*:ALL,GRANT"
        state: present
      with_items:
        - localhost
        - 127.0.0.1
        - "::1"
        - "{{ ansible_hostname }}"
        - "{{ inventory_hostname }}"

    - name: Remove anonymous MySQL users
      mysql_user:
        name: ''
        host_all: yes
        state: absent

    - name: Remove MySQL test database
      mysql_db:
        name: test
        state: absent

    - name: Install PHP
      yum:
        name: php
        state: present

    - name: Install PHP-MySQL
      yum:
        name: php-mysql
        state: present

    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```

 - **Explanation of Playbook**:
	 - **Apache Installation and Configuration**: Installs Apache HTTP Server (`httpd`), starts the service, and enables it to start at boot.
	 - **MySQL Installation and Configuration**: Installs MariaDB (a drop-in replacement for MySQL), starts the service, enables it to start at boot, sets the root password, removes anonymous users, and removes the test database.
	 - **PHP Installation**: Installs PHP and the PHP MySQL extension (`php-mysql`).
	 - **Restart Apache**: Restarts the Apache service to ensure it loads the PHP module.


#### Step 4: Run Your Ansible Playbook

1. **Execute the Playbook**:
    
- Run the playbook using the following command:
```bash
ansible-playbook -i hosts setup_lamp.yml
```
- This command tells Ansible to use the `hosts` inventory file and run the tasks defined in the `setup_lamp.yml` playbook on the servers listed under the `[lamp_servers]` group.

2. **Verify the LAMP Stack Installation**:

- After the playbook completes successfully, verify that Apache, MySQL, and PHP are installed and running on your servers.
- You can check if Apache is running by navigating to the IP address of one of your servers in a web browser. You should see the default Apache HTTP Server page.
- To verify PHP, you can create a simple PHP file (e.g., `info.php`) in the Apache document root (`/var/www/html` by default) with the following content:

```php
<?php phpinfo(); ?>
```

- - Access this file via a web browser to confirm PHP is functioning correctly.
3. **Test MySQL**:
    - Log in to the MySQL server using the root password you set in the playbook:
```bash
mysql -u root -p
```

- Verify that you can execute basic SQL commands.

#### Step 5: Customize and Extend Your Ansible Project

- **Add More Roles**: To modularize your playbook further, you can break down tasks into roles, such as separate roles for Apache, MySQL, and PHP.
- **Secure MySQL Further**: You can add more security configurations, such as binding MySQL to localhost only or setting up more secure user permissions.
- **Deploy Web Applications**: Extend your playbook to deploy a web application by copying the application files to the servers and setting up the database.

### Conclusion

You've now set up a basic LAMP stack using Ansible, automating the process of configuring web servers to serve PHP applications. Ansible playbooks provide a straightforward and powerful way to manage and automate infrastructure setup.

Feel free to modify the playbook as needed and add more functionality based on your project requirements. If you have any more questions or need further assistance, let me know!
