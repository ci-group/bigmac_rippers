How to connect to a ripper(CI Group computational servers)

- Create an ssh key https://phoenixnap.com/kb/ssh-with-key
- Send the ssh public key to Aart Stuurman
- Get an account on a ripper from Aart Stuurman
- Add the following file contents to ~/.ssh/config. Fill in your own ripper username:
```
AddKeysToAgent yes

Host bigmac
Hostname 145.108.195.17
Port 22
User hopper
ServerAliveInterval 10

Host ripper1
Hostname 10.0.0.1
Port 22
User <your_ripper_user>
ProxyJump bigmac

Host ripper2
Hostname 10.0.0.2
Port 22
User <your_ripper_user>
ProxyJump bigmac

Host ripper3
Hostname 10.0.0.3
Port 22
User <your_ripper_user>
ProxyJump bigmac

Host ripper4
Hostname 10.0.0.4
Port 22
User <your_ripper_user>
ProxyJump bigmac

Host ripper5
Hostname 10.0.0.5
Port 22
User <your_ripper_user>
ProxyJump bigmac

Host ripper6
Hostname 10.0.0.6
Port 22
User <your_ripper_user>
ProxyJump bigmac

Host ripper7
Hostname 10.0.0.7
Port 22
User <your_ripper_user>
ProxyJump bigmac
```

- you can now log into your ripper using `ssh ripper#`
