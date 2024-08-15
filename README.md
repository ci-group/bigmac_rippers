How to connect to a ripper(CI Group computational servers)

- Create an ssh key https://phoenixnap.com/kb/ssh-with-key
- Send the ssh public key to Ting-Chia(Karen) Chiang
- Get an account on a ripper from Ting-Chia(Karen) Chiang
- Add the following file contents to ~/.ssh/config. Fill in your own ripper username:
```
AddKeysToAgent yes

Host bigmac
Hostname 145.108.195.17
Port 22
User hopper
ServerAliveInterval 10

Host ripper#
Hostname 10.0.0.#
Port 22
User <your_ripper_user>
ProxyJump bigmac

```

- you can now log into your ripper using `ssh ripper#`
