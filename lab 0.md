# WireShark Installation 
- 1) install wireshark from google
  2) let your computer restart after installation
# Capture Packets: access NUS Homepage  (https://nus.edu.sg/)
<img width="959" height="503" alt="image" src="https://github.com/user-attachments/assets/31ed7488-25ec-4a87-bc39-fdc912032185" />

## What is the IP Address and Port of the NUS Homepage
- IP Address: 45.60.35.225, Port: 443

## What is the IP Address of PC when initially accessing the webpage 
- IP Address: 192.168.0.153, Port: 56301

## What is the process of the TCP three-way handshake
- SYN: request to establish TCP connection
<img width="953" height="503" alt="image" src="https://github.com/user-attachments/assets/887480fc-634e-4c7c-86d0-f92ff22c7fa8" />
- SYN - ACK: server acknowledges request and establishes connection
<img width="956" height="497" alt="image" src="https://github.com/user-attachments/assets/11bc1d40-9886-45f9-af21-cd4755d60aec" />
- ACK: TCP connection is established
<img width="958" height="506" alt="image" src="https://github.com/user-attachments/assets/99aeca39-c71a-41a9-ada6-8651867c8064" />

# Use Filter dns to find a DNS Packet 
<img width="959" height="503" alt="image" src="https://github.com/user-attachments/assets/297ef701-8294-4ff4-89a5-b984f2f88ec5" />

## What is the IP Address and Port of the DNS Server
- IP Address: 2001:4546:1::1, Port: 53

## What is the domain name in the query
- news.nus.edu.sg

## What Protocol(s) did this DNS Packet Use
<img width="928" height="348" alt="image" src="https://github.com/user-attachments/assets/8214e345-74a7-47f8-b933-961d556a5db7" />
- Layer 2 (Link): Ethernet II
- Layer 3 (Network): Internet Protocol Version 6 (IPv6)
- Layer 4 (Transport): Transmission Control Protocol (TCP)
- Later 5 (Application): Domain Name System (DNS)

# Access a HTTP Page

## HTTP Page I Used
- http://www.gzxyzn.com/Article/bjrk2/1644.html

## IP Address and Port of the Server Hosting the Page
<img width="957" height="503" alt="image" src="https://github.com/user-attachments/assets/ce1c4775-46c3-47e7-be39-4c6356e79298" />
- IP Address: 61.183.8.129, Port: 80

## What is the Request Method
<img width="959" height="505" alt="image" src="https://github.com/user-attachments/assets/fd439c4a-8fa2-4d4b-851a-54276dc0831f" />
- GET

## What is the response status code and what does it mean
- The server successfully received and processed your HTTP request and returned the requested webpage 


