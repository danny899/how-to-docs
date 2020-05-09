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

### Make sure you have two SSH keys in your ~/.ssh/ folder
```sh
~/.ssh/id_rsa  for GitHub account A
~/.ssh/id_rsa_account_b   for GitHub account B
```


## Add SSH Keys to GitHub Accounts

## Creating a SSH configuration for each host

## Switch between different SSH Keys in SSH-Agent

## Test the GitHub Setup

