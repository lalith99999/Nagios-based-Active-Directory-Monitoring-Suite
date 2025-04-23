# Project 1: Active Directory Health Checks with Nagios

## Overview
Nagios is an open-source monitoring system that helps identify and resolve IT infrastructure issues before they impact critical business services. This project demonstrates how to monitor the health and performance of Active Directory (AD) services using Nagios.

## Prerequisites
- **Windows Server** with Active Directory Domain Services (AD DS)
- **Nagios Core** installed on a monitoring server
- **NSClient++** installed on AD DS servers for monitoring

## Setup Instructions

### 1. Install Nagios Core
1. Download and install Nagios Core from the official website.
2. Follow the instructions specific to your operating system.
3. Start Nagios:
    ```bash
    sudo systemctl start nagios
    ```
4. Enable Nagios to start on boot:
    ```bash
    sudo systemctl enable nagios
    ```
5. Access Nagios at `http://<your_server_ip>/nagios` and log in with the default credentials (`nagiosadmin`).

### 2. Install NSClient++
1. Download NSClient++ from the official website.
2. Install NSClient++ on all AD DS servers.
3. Edit `nsclient.ini` to allow NRPE and collect necessary metrics:
    ```ini
    [/modules]
    NRPEListener = 1

    [/settings/NRPE/server]
    allow arguments = true
    insecure = true

    [/settings/NRPE/client]
    allowed hosts = <Nagios_Server_IP>

    [/settings/metrics]
    [/settings/log]
    ```

4. Restart NSClient++ service:
    ```powershell
    net stop nscp
    net start nscp
    ```

---

## Exercises

### Exercise 1: Adding AD DS Servers to Nagios
1. In the Nagios configuration file (`/usr/local/nagios/etc/nagios.cfg`), include the new configuration directory:
    ```cfg
    cfg_dir=/usr/local/nagios/etc/servers
    ```

2. Create a new configuration file for the AD DS servers (`/usr/local/nagios/etc/servers/adds.cfg`) with the following setup:
    ```cfg
    define host {
        use                     windows-server
        host_name               adds-server
        alias                   AD DS Server
        address                 <AD_Server_IP>
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     CPU Load
        check_command           check_nrpe!check_cpu
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     Memory Usage
        check_command           check_nrpe!check_memory
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     AD Logon Events
        check_command           check_nrpe!check_eventlog!application!warning!error!4624
    }
    ```

3. Restart Nagios:
    ```bash
    sudo systemctl restart nagios
    ```

**Expected Result:**  
Nagios should now monitor the AD DS servers and their services.

### Exercise 2: Monitoring AD Logon Events
1. Ensure that NSClient++ is configured to collect Windows Event Logs.
2. Add the following command definition to `commands.cfg` in the Nagios configuration directory:
    ```cfg
    define command {
        command_name    check_eventlog
        command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c check_eventlog -a 'application' 'warning' 'error' '$ARG1$'
    }
    ```

3. Modify the service definition in `adds.cfg` to monitor logon events:
    ```cfg
    define service {
        use                     generic-service
        host_name               adds-server
        service_description     AD Logon Events
        check_command           check_eventlog!4624
    }
    ```

**Expected Result:**  
Nagios should display logon events for the AD DS server.

### Exercise 3: Setting Up Alerts for AD Performance Issues
1. Edit the Nagios contacts configuration (`/usr/local/nagios/etc/objects/contacts.cfg`) to configure email notifications.
2. Define alert conditions in `adds.cfg`:
    ```cfg
    define service {
        use                     generic-service
        host_name               adds-server
        service_description     CPU Load
        check_command           check_nrpe!check_cpu
        notifications_enabled   1
        contacts                nagiosadmin
    }
    ```

3. Restart Nagios:
    ```bash
    sudo systemctl restart nagios
    ```

**Expected Result:**  
Nagios will now send email alerts for potential AD performance issues.

### Exercise 4: Creating a Dashboard for AD Metrics
1. Log in to the Nagios web interface.
2. Navigate to **Tactical Overview** for an overview of monitored services.
3. Click on **Service Details** to view detailed metrics.
4. Customize the view to create a dashboard for AD metrics.

**Expected Result:**  
A dashboard will display real-time AD metrics in the Nagios web interface.

### Exercise 5: Visualizing AD Security Metrics
1. Configure NSClient++ to collect security-related metrics if needed.
2. Add service definitions in `adds.cfg` to monitor security events like failed logons and account lockouts:
    ```cfg
    define service {
        use                     generic-service
        host_name               adds-server
        service_description     Failed Logon Attempts
        check_command           check_eventlog!4625
    }

    define service {
        use                     generic-service
        host_name               adds-server
        service_description     Account Lockouts
        check_command           check_eventlog!4740
    }
    ```

3. Restart Nagios:
    ```bash
    sudo systemctl restart nagios
    ```

 
Nagios will display key security metrics, offering a comprehensive view of AD security.



## Conclusion
This project demonstrates how to implement a complete Active Directory monitoring system with Nagios and NSClient++, helping you monitor the health, performance, and security of your AD DS servers.

---

This format should work well for GitHub documentation. Feel free to modify and structure it further based on your preferences or additional sections you'd like to include.
