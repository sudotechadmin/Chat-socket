# What Socket.IO is
Socket.IO is a library that enables low-latency, bidirectional and event-based communication between a client and a server.

The Socket.IO connection can be established with different low-level transports:
HTTP long-polling
WebSocket
WebTransport

Socket.IO will automatically pick the best available option, depending on:
the capabilities of the browser (see here and here)
the network (some networks block WebSocket and/or WebTransport connections)
Check out how it is working :- https://socket.io/docs/v4/how-it-works/

## Features
Multiple users can join a chat room by each entering a unique username on website load.
Users can type chat messages to the chat room.
A notification is sent to all users when a user joins or leaves the chatroom.
For more detailed documentation visit :- https://socket.io/docs/v4/tutorial/introduction

## Socket.IO Chatapp Server Implementation
Please check our documentation here:- https://socket.io/docs/v4/

## Prerequisites:
Node.js
npm
Docker and Minikube (Linux). 
Docker-desktop with Kubernetes enabled.(Windows).

## How to use locally
### 1. Clone the repo:-
    $ git clone https://github.com/akbhatisain/Chat-Socket.io
### 2. Switch to project directory:
    $ cd Chat-Socket.io
### 3. Setup the project:
    $ npm i
    $ npm start
4. Test your environment :- Search on your browser http://localhost:3000.
5. Once you finish with testing Stop the server with Ctrl+c in terminal.

## Dockerize the Application
### 1. Create a Docker Image
    $ docker build -t chatapp:v1 ./

### 2. Run the container 
    $ docker run -itd -p 3001:3000 --name chat-app chatapp:v1

### 3. Checking the container:-
    $ Docker ps

### 4. Details about the container
    $ docker container inspect <CONTAINER ID>

5. Test your environment :- Search on your browser http://localhost:3000. OR http://127.0.0.1:3000

### 6. Once you finish with testing destroy the container.
    $ docker rm <CONTAINER ID>
