import socket
import threading

# Server setup
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
host = '127.0.0.1'  # Localhost
port = 55555
server.bind((host, port))
server.listen()

clients = []
nicknames = []

# Function to broadcast messages to all clients
def broadcast(message):
    for client in clients:
        client.send(message)

# Function to handle individual client connections
def handle(client):
    while True:
        try:
            message = client.recv(1024)
            broadcast(message)
        except:
            index = clients.index(client)
            clients.remove(client)
            client.close()
            nickname = nicknames[index]
            broadcast(f'{nickname} left the chat!'.encode('ascii'))
            nicknames.remove(nickname)
            break

# Function to receive and accept client connections
def receive():
    while True:
        client, address = server.accept()
        print(f"Connected with {str(address)}")

        client.send('NICK'.encode('ascii'))
        nickname = client.recv(1024).decode('ascii')
        nicknames.append(nickname)
        clients.append(client)

        print(f"Nickname of the client is {nickname}")
        broadcast(f"{nickname} joined the chat!".encode('ascii'))
        client.send('Connected to the server!'.encode('ascii'))

        thread = threading.Thread(target=handle, args=(client,))
        thread.start()

print("Server is listening...")
receive()
import socket
import threading

# Client setup
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
host = '127.0.0.1'  # Localhost
port = 55555
client.connect((host, port))

nickname = input("Choose your nickname: ")

# Function to receive messages from the server
def receive():
    while True:
        try:
            message = client.recv(1024).decode('ascii')
            if message == 'NICK':
                client.send(nickname.encode('ascii'))
            else:
                print(message)
        except:
            print("An error occurred!")
            client.close()
            break

# Function to send messages to the server
def write():
    while True:
        message = f'{nickname}: {input("")}'
        client.send(message.encode('ascii'))

# Start threads for receiving and sending messages
receive_thread = threading.Thread(target=receive)
receive_thread.start()

write_thread = threading.Thread(target=write)
write_thread.start()
