# Soc-Home-Lab-
SOC home lab: SSH brute force detection using Splunk SIEM, Universal Forwarder, and Wireshark on Apple Silicon
# SOC Home Lab - SSH Brute Force Detection with Splunk

## Overview

I built this home lab to get hands-on experience with a real SIEM pipeline - ingesting logs, writing detection queries, and setting up alerts. The goal was to simulate an SSH brute force attack and catch it using Splunk.

The setup runs on an Apple Silicon M5 Mac using VirtualBox with an Ubuntu 22.04 ARM64 VM. Splunk Enterprise is installed natively on the Mac, and a Universal Forwarder on the Ubuntu VM ships authentication logs over to Splunk in real time.

## What I Built

The Ubuntu VM acts as the target machine. Its auth logs get forwarded to Splunk on the Mac via the Universal Forwarder on port 9997. From there I can search, visualize, and alert on anything in those logs.

I also installed Wireshark on the Ubuntu VM to capture live network traffic on the bridged network interface.

## Detection - Failed SSH Logins

To find failed login attempts in Splunk I used this SPL query:

    index=main source="/var/log/auth.log" "Failed password"

This surfaces log entries that look like this:

    2026-04-08T17:03:08 ubuntulab sshd[4472]: Failed password for nix from 127.0.0.1 port 52450 ssh2

## Brute Force Alert

I set up a scheduled alert in Splunk that runs every 15 minutes and checks for more than 5 failed logins in that window. The SPL behind it is:

    index=main source="/var/log/auth.log" "Failed password" | stats count by host

The alert is set to High severity and logs to Triggered Alerts when it fires.

## Attack Simulation

To test the detection I SSHed into localhost from inside the Ubuntu VM and intentionally entered the wrong password six times. The failed attempts showed up in Splunk on the next polling cycle exactly as expected.

## Troubleshooting I Worked Through

Getting everything connected on Apple Silicon was not straightforward. The biggest issue was that the Ubuntu VM started on NAT networking, which meant the forwarder could not reach Splunk on the Mac. Switching the VM to Bridged networking gave it a real IP on the home network and fixed the connection.

Other things I ran into along the way - Splunk has no native ARM64 Docker image so I had to install it directly on the Mac instead, the VM disk needed to be expanded manually using lvextend, and log rotation caused the forwarder to lose its place in auth.log which required clearing the fishbucket and restarting.

## Skills Practiced

Working through this lab gave me real practice with Splunk administration, SPL query writing, Linux log management, alert configuration, network packet capture with Wireshark, and virtualization on Apple Silicon.

## What's Next

I plan to add IDS capabilities with Snort or Suricata, pull in firewall logs, and build a live Splunk dashboard for monitoring. On the certification side I am working toward CompTIA Security+.

Built alongside the Google Cybersecurity Professional Certificate and Loyola University New Orleans Cybersecurity Program.
