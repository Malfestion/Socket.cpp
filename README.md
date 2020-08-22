Easy C++ Socket Class
=====================

**Simple Socket Echo Server**
Can also be combined with lists and threads to run multiple socket connections at the same time.
In this example the server accepts only one client.
But if you store the sockets and iterate through them (or split them with threads in multiple processes) you can accept multiple clients.
```cpp
string ip = "localhost";
string port = "1234";
Socket *masterSocket = new Socket(AF_INET,SOCK_STREAM,0); //AF_INET (Internet mode) SOCK_STREAM (TCP mode) 0 (Protocol any)
int optVal = 1;
masterSocket->socket_set_opt(SOL_SOCKET, SO_REUSEADDR, &optVal); //You can reuse the address and the port
masterSocket->bind(ip, port); //Bind socket on localhost:1234

masterSocket->listen(10); //Start listening for incoming connections (10 => maximum of 10 Connections in Queue)

while (true) {
    vector<Socket> reads(1);
    reads[0] = *masterSocket;
    int seconds = 10; //Wait 10 seconds for incoming Connections
    if(Socket::select(&reads, NULL, NULL, seconds) < 1){ //Socket::select waits until masterSocket reveives some input (for example a new connection)
        //No new Connection
        continue;
    }else{
        //Something happens, let's accept the connection
        break;
    }
}
Socket *newSocket = masterSocket->accept(); //Accept the incoming connection and creates a new Socket to the client

while (true) {
    vector<Socket> reads(1);
    reads[0] = *newSocket;
    int seconds = 10; //Wait 10 seconds for input
    if(Socket::select(&reads, NULL, NULL, seconds) < 1){ //Socket::select waits until masterSocket reveives some input (for example a message)
        //No Input
        continue;
    }else{
        string buffer;
        newSocket->socket_read(buffer, 1024); //Read 1024 bytes of the stream
        
        newSocket->socket_write(buffer); //Sends the input back to the client (echo)
    }
}
newSocket->socket_shutdown(2);
newSocket->close();

masterSocket->socket_shutdown(2);
masterSocket->close();
```

**Simple Socket Client**
```cpp
string ip = Socket::ipFromHostName("google.com"); //Get ip addres from hostname
string port = "80"; //let's talk on http port
Socket *sock = new Socket(AF_INET,SOCK_STREAM,0);  //AF_INET (Internet mode) SOCK_STREAM (TCP mode) 0 (Protocol any)
sock->connect(ip, port); //Connect to google.com:80
sock->socket_write("GET / HTTP/1.1\r\n\r\n");//Send GET request to google.com:80
int seconds = 10;//Wait 10 second for response
vector<Socket> reads(1);
reads[0] = *sock;
if(sock->select(&reads, NULL, NULL, seconds) < 1){//Socket::select waits until sock reveives some input (for example the answer from google.com)
    //Something went wrong
}else{
    string buffer;
    sock->socket_read(buffer, 1024);//Read 1024 bytes of the answer
    cout << buffer << endl;
}
```

**Simple UDP multithread message passing example**
```cpp
#include <thread>
#include <mutex>
#include <iostream>
#include "Socket.h"

using namespace std;

string ip= "localhost";
string port= "5001";
mutex inputM;
mutex outputM;

void serverUDP(){
  Socket *serverSocket= new Socket(AF_INET,SOCK_DGRAM,0);
  serverSocket->bind(ip,port);
  while(1){
    string buffer;
    serverSocket->socket_readFrom(buffer,10240,ip,port);
    outputM.lock();
    cout<<"El servidor Recibio:"<<buffer<<endl;
    outputM.unlock();
  }
}
void clientUDP(int id){
  string userIn;
  Socket *clientSocket= new Socket(AF_INET,SOCK_DGRAM,0);
  while(1){
    inputM.lock();
    getline(cin,userIn);
    inputM.unlock();
    clientSocket->socket_writeTo(userIn,ip,port);
    outputM.lock();
    cout<<"se envio el mensaje:\""<<userIn<<"\" Sender=cliente "<<id<<endl;
    outputM.unlock();
  }
}
int main(){
thread t1(serverUDP);//se crea un hilo y se envia a serverUDP() con sus parametros
thread t2(clientUDP,1);//se crea un hilo y se envia a clientUDP() con sus parametros
thread t3(clientUDP,2);//se crea un hilo y se envia a clientUDP() con sus parametros
t1.join();//se espera a que los hilos terminen
t2.join();//se espera a que los hilos terminen
//t3.join();//se espera a que los hilos terminen
return 0;

  printf("1\n");
  return 0;
}
```
