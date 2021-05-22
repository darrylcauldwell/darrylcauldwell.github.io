+++
title = "VSCode Remote SSH"
date = "2021-05-13"
description = "VSCode Remote SSH"
tags = [
    "macos"
]
categories = [
    "handy hacks"
]
thumbnail = "clarity-icons/code-144.svg"
+++

The more I use Visual Studio the more features and extensions I find which really help my workflow.

I tend to work with many headless remote systems from Windows jump servers. I've traditionally used PuTTY and MTPuTTY (Multi-Tabbed PuTTY) as SSH client and CyberDuck or WinSCP to copy files via SFTP over SSH. The VS Code 'Remote - SSH' extension allows you to open a remote folder on any remote machine, virtual machine, or container with a running SSH server and take full advantage of VS Code's feature set. Once connected to a server, you can interact with files and folders anywhere on the remote filesystem.

This extension makes managing multiple headless remote systems reallly painless!

However, while most Linux servers connect without issue some would fail with message like “open failed: administratively prohibited: open failed”. In order for VS Code to be able to connect successfully to the Linux VM, we need to make sure that AllowTcpForwarding is enabled.  To enable this edit the `/etc/ssh/sshd_config` and ensure `AllowTcpForwarding yes` and restart SSHD `systemctl restart sshd`.