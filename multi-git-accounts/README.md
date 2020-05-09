# How to setup multiple github accounts in the same computer

Quite often we have the need to use multiple github accounts in the same machine, being able to easily switch between different github accounts and work on multiple projects in parallel is a very common use case.

In this document, I'm going to show people how I setup multiple github accounts on my Mac. I will two projects project-A and project-B as an example, and here are the two sample project links: 
```sh
Project A:  git@github.com:account1/project-A.git
Project B:  git@github.com:account2/project-B.git
```

## Generate SSH Keys
First we need to generate two SSH keys, one for each account. If you already have a SSH, you can use it for one of the GitHub account, in this tutorial, i assume we don't have any SSH setup yet.

### Run the following command to generate the first/default SSH key. 
```sh
ssh-keygen -t rsa
```
Once run, you should have the following two files in your machine. You can use this key for the project you spend most time on. I'm going to use this key for project A in this tutorial.
```sh
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

### Now generate the second SSH key, using the following command
```sh
ssh-keygen -t rsa -f "id_rsa_account_b" -C "your@email.com"
```
This should create a file in the `~/.ssh/` folder with name `id_rsa_account_b` and `id_rsa_account_b.pub`

### Make sure you have two SSH keys in your `~/.ssh/` folder
```sh
~/.ssh/id_rsa.pub  // We going to use for GitHub account A
~/.ssh/id_rsa_account_b.pub   //for GitHub account B
```


## Add SSH Keys to GitHub Accounts
Now let's upload/copy the SSH keys into your GitHub accounts. Here is the information on [how to upload your SSH key onto GitHub](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account).

1. Copy the SSH to clipboard
```sh
pbcopy < ~/.ssh/id_rsa.pub
```
2. In the upper-right corner of any page, click your profile photo, then click *Settings*. 
3. Click **SSH and GPG keys**
4. Click **New SSH key** or **Add SSH key**.
5. In the "Title" field, add a descriptive label for the new key. For example, if you're using a personal Mac, you might call this key "Personal MacBook Air".
6. Paste your key into the "Key" field. 
7. Click **Add SSH Key**
8. If prompted, confirm your GitHub password. 

Repeat the same steps on your second SSH key `~/.ssh/id_rsa_account_b.pub`, and make sure it's uploaded to your second account (GitHub Account B).

## Creating a SSH configuration for each host
We will use a SSH configuration file for domain identity resolution. Run the following command to create or open your SSH configuration file.
```sh
sudo vi ~/.ssh/config
```

Now paste in the following content
```sh
# GitHub Account A config
Host github.com-a
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa
   
# GitHub Account B config
Host github.com-b   
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa_account_b
```
Basically the above configuration is telling ssh-agent to:
* Use the `~/.ssh/id_rsa` key for any Git URL with domain github.com-a
* Use the `~/.ssh/id_rsa_account_b` key for any Git URL with domain github.com-b

## Clone the GitHub Repo
Once the SSH keys are setup, we are now ready to clone/check out the github repositories.

### First, run the following command to switch ssh-agent to use your first key
```sh
ssh-add -D
ssh-add ~/.ssh/id_rsa
```

Then run the git clone command to clone the project(s) in your first github account.
```sh
git clone git@github.com-a:account1/project-A.git
```
Note the domain name of the repo is **github.com-a**, this is consistent with your git configuration in step 3.

### Now, switch to your second GitHub account using ssh key
```sh
ssh-add -D
ssh-add ~/.ssh/id_rsa_account_b
```

Then run the git clone command to clone the project(s) in your second github account.
```sh
git clone git@github.com-b:account1/project-B.git
```
Note the domain name of the repo is **github.com-b**

## Switch between different SSH Keys in SSH-Agent
To switch between different GitHub accounts, you simple switch your SSH key using the following command
```sh
ssh-add -D
ssh-add ~/.ssh/id_rsa_account_b  //This changes according to your GitHub SSH key
```

If you have issue with the SSH key switch, make sure your SSH agent is running, run `eval "$(ssh-agent -s)"`

## Done.
Now you have both GitHub accounts working on the same machine. Feel free to leave any comment if you have hit issues.
