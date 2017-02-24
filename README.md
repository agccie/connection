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
 
 The result of the cmd() function is a string name representing what was matched by the command.  For most operations, this will be 'prompt', however custom match options can be provided to the cmd() function for more flexibility.  The output from each command is saved in the __output__ variable. 
 
 ```python
     def cmd(self, command, **kargs):
        """
        execute a command on a device and wait for one of the provided matches to return.
        Required argument string command
        Optional arguments:
            timeout - seconds to wait for command to completed (default to self.timeout)
            sendline - boolean flag to use send or sendline fuction (default to true)
            matches - dictionary of key/regex to match against.  Key corresponding to matched
                regex will be returned.  By default, the following three keys/regex are applied:
                    'eof'       : pexpect.EOF
                    'timeout'   : pexpect.TIMEOUT
                    'prompt'    : self.prompt
            echo_cmd - boolean flag to echo commands sent (default to false)
                note most terminals (i.e., Cisco devices) will echo back all typed characters
                by default.  Therefore, enabling echo_cmd may cause duplicate cmd characters
        Return:
        returns the key from the matched regex.  For most scenarios, this will be 'prompt'.  The output
        from the command can be collected from self.output variable
        """
 ```
 
# Examples
 
* Basic example for ssh into a device, executing a command, and viewing the output.

```python
        c = Connection("host1")
        c.username = "admin"
        c.password = "cisco"
        c.cmd("terminal length 0")
        c.cmd("show version")
        print "version of code: %s" % c.output    
```

* Setting the logfile to stdout

```python
         import sys
         from connection import Connection
         c = Connection("host1")
         c.log = sys.stdout
```

* Matching a custom attribute
There are many operations where a device may prompt the user for confirmation instead of returning the prompt.  An example reload an IOS/NXOS device:

           fab3-leaf103# reload
           This command will reload the chassis, Proceed (y/n)? [n]:
 
 For these operations, the cmd() function should include a match dictionary to explicitly look for the y/n prompt:
 
 ```python
            from connection import Connection
            c = Connection("esc-aci-fab3")
            if c.login():     
                m = {
                    "confirm": "\[n\]"  # regex match for '[n]' string
                }
                result = c.cmd("reload", matches=m)
                if result == "prompt":
                    print "the prompt was returned..."
                elif result == "confirm":
                    print "the confirm option was returned!  Sending 'y'"
                    result = c.cmd("y", timeout=5)
                    if result == "eof" or result == "timeout":
                        print "looks like the box is reloading!"
```

* ssh into a non-cisco device

 The connection script is using pexpect and is looking for a prompt to determine when the login is successful.  To login to a different device, simply set the prompt to a regex matching the devices prompt.  For example, when access a ubuntu host, you may see the following prompt:
 
         agccie@ag-docker2:~$
 
 The prompt needs to match the string :~$.  An appropriate regex might be "[^:]:~\$[ ]*$".  When setting up the connection, update the prompt before login:

 ```python
         from connection import Connection
         c = Connection("host1")
         c.username = agccie
         c.password = cisco
         c.prompt = "[^:]:~\$[ ]*$"
         print "Login successful: %r" % c.login() 
```       
