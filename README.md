# Jaringan-Computer

# Langkah-langkah menjalankan Proyek
# 1. Persiapkan Raspberry Pi
Sebelum menjalankan script, pastikan Rasbberry Pi anda sudah terinstal di Python 3, dengan menggunakan perintah berikut
python3 --version
buka Terminal pada Rasberry Pi.
Buat File baru untuk menyimpan script python dengan menggunakan perintah berikut:

Copy Code

nano petrus.py

salin dan tempelkan code dibawah ini  ke dalam file petrus.py:

python

Copy code


from socket import *

import sys
if len(sys.argv) <= 1:
    print('Usage : "python ProxyServer.py server_ip"\n[server_ip : It is the IP Address Of Proxy Server]')
    sys.exit(2)

Create a server socket, bind it to a port, and start listening
tcpSerSock = socket(AF_INET, SOCK_STREAM)
tcpSerSock.bind((sys.argv[1], 7000))
tcpSerSock.listen(5)

while True:
    # Start receiving data from the client
    print('Ready to serve...')
    tcpCliSock, addr = tcpSerSock.accept()
    print('Received a connection from:', addr)
    
    # Receive message from client
    message = tcpCliSock.recv(1024).decode()
    print(message)
    
    # Extract the filename from the given message
    if len(message.split()) > 1:
        print(message.split()[1])
        filename = message.split()[1].partition("/")[2]
        print(filename)
        fileExist = "false"
        filetouse = "/" + filename
        print(filetouse)
        
        try:
            # Check whether the file exists in the cache
            f = open(filetouse[1:], "r")
            outputdata = f.readlines()
            fileExist = "true"
            
            # ProxyServer finds a cache hit and generates a response message
            tcpCliSock.send("HTTP/1.0 200 OK\r\n".encode())
            tcpCliSock.send("Content-Type:text/html\r\n".encode())
            # Send cached data
            for line in outputdata:
                tcpCliSock.send(line.encode())
            print('Read from cache')
        
        except IOError:
            if fileExist == "false":
                # Create a socket on the proxy server
                c = socket(AF_INET, SOCK_STREAM)
                hostn = filename.replace("www.", "", 1)
                print(hostn)
                
                try:
                    # Connect to the socket to port 80
                    c.connect((hostn, 80))
                    
                    # Create a temporary file on this socket and ask port 80 for the file requested by the client
                    fileobj = c.makefile('r', 0)
                    fileobj.write("GET /" + filename + " HTTP/1.0\n\n")
                    
                    # Read the response into buffer
                    buffer = c.recv(1024)
                    
                    # Create a new file in the cache for the requested file.
                    # Also send the response in the buffer to client socket and the corresponding file in the cache
                    tmpFile = open("./" + filename, "wb")
                    while len(buffer) > 0:
                        tcpCliSock.send(buffer)
                        tmpFile.write(buffer)
                        buffer = c.recv(1024)
                    tmpFile.close()
                
                except:
                    print("Illegal request")
                
                c.close()
    
    # HTTP response message for file not found
    else:
        tcpCliSock.send("HTTP/1.0 404 Not Found\r\n".encode())
        tcpCliSock.send("Content-Type:text/html\r\n\r\n".encode())
        tcpCliSock.send("<html><body><h1>404 Not Found</h1></body></html>".encode())
    
    # Close the client connection
    tcpCliSock.close()
Setelah itu code tersebut disimpan dan untuk memanggil nya menggunakan perintah berikut:
python3 petrus.py 192.168.5.83
Setelah itu untuk bisa dicek apakah requrest server sudah masuk atau belum, dalam code diatas saya menggunakan port 7000
cek di Browser: 192.168.5.83:7000 maka akan terlihat pesan yang masuk di ip tersebut
