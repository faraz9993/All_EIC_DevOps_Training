# Task:10 Shell Script
### In this task, I have written shell script that gives the collected information of the CPU usage, memory utilization, disk space, network statistics, system logs with event of critical and errors, status of essential services such as Apache, nginx and MySQL. The output all these things will be saved in a file named output.txt.

### Not only this, I have also scheduled the cron job for this script as per which the script can run at every 5 minutes.

```
#!/bin/bash
 
echo "
 
-------------------------------------------" >> output.txt

top -b -n 1 | head -n 35 >> output.txt
 
 
echo "-------------------------------------------
 
	" >> output.txt

cat /var/log/syslog | grep "warnin" | tail -20 >> output.txt
cat /var/log/syslog | grep error | tail -20 >> output.txt
 
 
echo "-------------------------------------------
 
	" >> output.txt

echo "The status of nginx service is: " >> output.txt
systemctl status nginx.service | grep Active >> output.txt

echo "The status of apache2 service is: " >> output.txt
systemctl status apache2.service | grep Active >> output.txt

echo "-------------------------------------------

        " >> output.txt
```

### You can schedule the cron job to run this script at every 5 minutes using below command:
```
crontab -e
```

### After running this command, a file will be opened where you have to insert the below content:

```
*/1     *       *       *       *       sh /home/einfochips/"All Extras"/EIC_DevOps_Training/Shell_Task/shell.sh
```




