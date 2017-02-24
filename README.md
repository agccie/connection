# connection
Python SSH/Telnet connection script for automating CLI

# Description
There are several libraries available within python that can be used to connect and manage devices. The pexpect module is an exceptionally useful module because it allows you to utilize must programs already available on the system where the script is executed. For example, you can use pexpect to spawn a telnet, ssh, or ftp session without relying on custom python libraries for telnet/ssh/ftp.  This module abstracts some of the complexities of working with expect so users can programmatically do quick CLI functions 

# Installation

* Python 2.7+

      git clone https://github.com/agccie/connection.git
      more
      
# Usage

        hostname         (req) hostname or IP address
        username         (opt) username credential (default 'admin')
        password         (opt) password credential (default 'cisco')
        enable_password  (opt) enable password credential (IOS only) (default 'cisco')
        protocol         (opt) telnet/ssh option (default 'ssh')
        port             (opt) port to connect on (if different from telnet/ssh default)
        timeout          (opt) wait in seconds between each command (default 30)
        prompt           (opt) prompt to expect after each command (default for IOS/NXOS)
        log              (opt) logfile (default None)
        verify           (opt) verify/enforce strictHostKey values for SSL (disabled by default)
        searchwindowsize (opt) maximum amount of data used in matching expressions
                               extremely important to set to a low value for large outputs
                               pexpect default = None, setting this class default=256
        force_wait       (opt) some OS ignore searchwindowsize and therefore still experience high
                               CPU and long wait time for commands with large outputs to complete.
                               A workaround is to sleep the script instead of running regex checking
                               for prompt character.
                               This should only be used in those unique scenarios...
                               Default is 0 seconds (disabled).  If needed, set to 8 (seconds)

        functions:
        connect()        (opt) connect to device with provided protocol/port/hostname
        login()          (opt) log into device with provided credentials
        close()          (opt) close current connection
        cmd()            execute a command on the device (provide matches and timeout)
        
 # Examples
 
 # FAQ
 
 ## How do I log to standard out
 The log option supports a filename or file pointer.  To log to standard out, you can just set log to sys.stdout.  For example:
         
         import sys
         from connection import Connection
         c = Connection("host1")
         c.log = sys.stdout
 
 ## How do I ssh into a non-cisco device
 The connection script is using pexpect and is looking for a prompt to determine when the login is successful.  To login to a different device, simply set the prompt to a regex matching the devices prompt.  For example, when access a ubuntu host, you may see the following prompt:
 
         agccie@ag-docker2:~$
 
 The prompt needs to match the string :~$.  An appropriate regex might be "[^:]:~\$[ ]*$".  When setting up the connection, update the prompt before login:
 
         from connection import Connection
         c = Connection("host1")
         c.username = agccie
         c.password = cisco
         c.prompt = "[^:]:~\$[ ]*$"
         print "Login successful: %r" % c.login()
 
       
