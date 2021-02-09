# Kdump配置-Ubuntu
https://ubuntu.com/server/docs/kernel-crash-dump 按照里面进行配置，写的十分详细
```bash
sudo apt install linux-crashdump
```
![avatar](https://github.com/LoganZhao/Azure/blob/master/%7B954FEAB6-D9F0-442D-8BE5-2A17709E4A72%7D.png.jpg)
![avatar](https://github.com/LoganZhao/Azure/blob/master/%7B09973EA7-B7BE-4B61-8574-5333D92754C3%7D.png.jpg)

# 配置audit reboot （适用于RH/CENTOS，同样适用于Ubuntu）
https://www.thegeekdiary.com/audit-rules-to-log-reboot-command-executions-in-centos-rhel/

Install auditd service if your system dont have:
```bash
sudo apt-get update
sudo apt-get install auditd
```
![avatar](https://github.com/LoganZhao/Azure/blob/master/%7BCEC09131-0637-4BEE-9DFB-64434BF7E4B5%7D.png.jpg)
