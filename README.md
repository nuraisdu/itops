# itops
here code related for automation process in  manuFACTURING
import paramiko
import logging

# Configure logging
logging.basicConfig(filename='update_log.log', level=logging.INFO, format='%(asctime)s %(message)s')

# List of systems to update (replace with actual IP addresses or hostnames)
systems = ['192.168.1.1', '192.168.1.2', '192.168.1.3']
username = 'user' # Replace with actual username
password = 'password' # Replace with actual password

def check_for_updates(ssh_client):
    """Check for software updates."""
    stdin, stdout, stderr = ssh_client.exec_command('sudo apt-get update && apt list --upgradable')
    updates = stdout.read().decode()
    if 'upgradable' in updates:
        return True
    else:
        return False

def install_updates(ssh_client):
    """Install software updates."""
    stdin, stdout, stderr = ssh_client.exec_command('sudo apt-get upgrade -y')
    result = stdout.read().decode() + stderr.read().decode()
    return result

def update_system(hostname, username, password):
    """Update software on a single system."""
    try:
        ssh_client = paramiko.SSHClient()
        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh_client.connect(hostname, username=username, password=password)
        
        if check_for_updates(ssh_client):
            result = install_updates(ssh_client)
            logging.info(f"Updates installed successfully on {hostname}: {result}")
        else:
            logging.info(f"No updates available for {hostname}.")
        
        ssh_client.close()
    except Exception as e:
        logging.error(f"Failed to update {hostname}: {str(e)}")

if __name__ == "__main__":
    for system in systems:
        update_system(system, username, password)
