---
layout: post
title: "SSH Configs and Multiple Github Accounts"
---

SSH is a pretty neat technology using asymmetric cryptography to keep your communication with the server safe and secure. If you've used Github with SSH keys, you'll know that you have to provide your SSH public key to your account's list of public keys. Then, when you clone/pull/push anything to your repo, it'll use your public key to open an SSH tunnel to transport your commands. You can verify your SSH connection with `ssh -T git@github.com`, and it should respond with `Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.`

That's fine and all, but the question I had was what do you do when you have multiple Github accounts that you'd like to switch between when doing work? For example, say I have a work account that I use, as well as a personal account for home projects. I would like to keep them separate but still use the same machine.

# (Extremely) Brief SSH overview
OpenSSH, your de facto SSH suite of tools on Linux, by default keeps your public and private keys inside your home directory in a dot directory called `.ssh`, unsurprisingly. If you take a look, you might see some files like `id_rsa`, `id_rsa.pub`, and maybe `config`. These are your private key, public key, and SSH configuration files respectively. One of the first rules of using SSH or any kind of asymmetric (or symmetric) cryptography of course is to __never__ share the private key, i.e. the one without the `.pub` ending. If you open it up, you'll see a __`-----BEGIN RSA PRIVATE KEY-----`__ to help you remember. What I like to do is generate an SSH key per client-server pair, so in case one is compromised, it won't affect the integrity of my other connections, e.g. `id_rsa_gitaccount1`, `id_rsa_gitaccount1.pub`, and so on. You can tell SSH which file to use with `ssh -i <private key file>`, but that's a tedious way of using it every single time you want to SSH into a different machine. Enter, the `config` file.

# SSH Config file
The config file stores information about how you want to set up an SSH connection to a particular server.
Here's a short example:

```bash
# This is my server
Host my-server
    Hostname myserver.com
    User iamuser
    IdentityFile ~/.ssh/my-server-key.pub
```

Here, `Host my-server` is your alias to the account you're SSHing to, with input parameters of your server hostname, username, and the public key file location. The manual way of SSHing into your server would be with a command like `ssh iamuser@myserver.com`, in which it'd ask you to provide the username/password if that's enabled, or allow/deny you based on the public key you provide.
However, with this configuration, you can just run `ssh my-server` to use your SSH identity file to acces your server, given that you've set one up.

# Using SSH config with Github
So how does this tie into using it with multiple Github acounts? You can add multiple Hosts to your SSH config file, and differentiate them with their respective Host identities!

```bash
# This is my personal Github
Host gh-personal
    Hostname github.com
    User git
    AddKeysToAgent yes
    IdentityFile ~/.ssh/gh-personal.pub

# This is my work Github
Host gh-work
    Hostname github.com
    User git
    AddKeysToAgent yes
    IdentityFile ~/.ssh/gh-work.pub
```

Here, we have added two identities to our config, one for our personal Github account, and one for our work account. The `AddKeysToAgent` parameter tells ssh-agent to add the private key used for authentication if it has not been added already. When we connect to our repo with our respective SSH identities, SSH will use that respective public key to access the repository.

When you clone a repo with SSH, you can direct git which SSH configuration to use. Cloning with `git clone git@github.com:a-github-account/a-repository` is saying to use whatever your SSH key is for user `git` at `github.com` to clone `a-repository`, assuming you have access rights. To direct SSH to use, for example, my personal public key file, I would need to change it to `git clone git@gh-personal:a-github-account/a-repository`. This will tell git to clone using my `gh-personal` SSH config.

If you need to ever tweak the SSH configuration used for a particular repo, you can access the `.git/config` file in the repository, and change the remote url to match the account you'd like to use from your SSH config file.

And that should be it! Now you have the power to use not only multiple Github accounts on the same machine, but also tweak and configure how you SSH into different machines.