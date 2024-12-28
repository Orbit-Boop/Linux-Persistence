#!/bin/bash

#echo -n "[sudo] password for kali: "
#read -p "[sudo] password for kali: " $pass
#echo 
#echo $pass

pass=kali123
sshpass -p "$pass" ssh -o 'StrictHostKeyChecking=no' seva@192.168.17.52 'id'
#sshpass -p $pass ssh seva@192.168.17.52 'touch Hello'
#sshpass -p $pass ssh seva@192.168.17.52 'ls -la'
