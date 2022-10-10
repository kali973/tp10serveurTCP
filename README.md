# tp10serveurTCP

mplémentation serveur-client UDP en C
Si nous créons une connexion entre le client et le serveur à l’aide de TCP, il a peu de fonctionnalités comme TCP est adapté aux applications qui nécessitent une grande fiabilité et le temps de transmission est relativement moins critique. Il est utilisé par d’autres protocoles comme HTTP, HTTPs, FTP, SMTP, Telnet. TCP réorganise les paquets de données dans l’ordre spécifié. Il y a une garantie absolue que les données transférées restent intactes et arrivent dans le même ordre dans lequel elles ont été envoyées. TCP effectue le contrôle de flux et nécessite trois paquets pour configurer une connexion socket, avant que des données utilisateur puissent être envoyées. TCP gère la fiabilité et le contrôle de la congestion. Il effectue également la vérification des erreurs et la récupération des erreurs. Les paquets erronés sont retransmis de la source vers la destination.

L’ensemble du processus peut être décomposé en étapes suivantes :


L’ensemble du processus peut être décomposé en étapes suivantes :

Serveur TCP –

    en utilisant create(), Créer une socket TCP.
    en utilisant bind(), liez le socket à l’adresse du serveur.
    en utilisant listen(), mettez le socket du serveur en mode passif, où il attend que le client s’approche du serveur pour établir une connexion
    en utilisant accept(), À ce stade, la connexion est établie entre le client et le serveur, et ils sont prêts à transférer des données.
    Revenez à l’étape 3.

Client TCP –

    Créer une socket TCP.
    connecter le socket client nouvellement créé au serveur.

Serveur TCP :
C

#include <stdio.h>
#include <netdb.h>
#include <netinet/in.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/types.h>
#define MAX 80
#define PORT 8080
#define SA struct sockaddr

// Function designed for chat between client and server.
void func(int connfd)
{
char buff[MAX];
int n;
// infinite loop for chat
for (;;) {
bzero(buff, MAX);

        // read the message from client and copy it in buffer
        read(connfd, buff, sizeof(buff));
        // print buffer which contains the client contents
        printf("From client: %s\t To client : ", buff);
        bzero(buff, MAX);
        n = 0;
        // copy server message in the buffer
        while ((buff[n++] = getchar()) != '\n')
            ;
   
        // and send that buffer to client
        write(connfd, buff, sizeof(buff));
   
        // if msg contains "Exit" then server exit and chat ended.
        if (strncmp("exit", buff, 4) == 0) {
            printf("Server Exit...\n");
            break;
        }
    }
}

// Driver function
int main()
{
int sockfd, connfd, len;
struct sockaddr_in servaddr, cli;

    // socket create and verification
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1) {
        printf("socket creation failed...\n");
        exit(0);
    }
    else
        printf("Socket successfully created..\n");
    bzero(&servaddr, sizeof(servaddr));
   
    // assign IP, PORT
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    servaddr.sin_port = htons(PORT);
   
    // Binding newly created socket to given IP and verification
    if ((bind(sockfd, (SA*)&servaddr, sizeof(servaddr))) != 0) {
        printf("socket bind failed...\n");
        exit(0);
    }
    else
        printf("Socket successfully binded..\n");
   
    // Now server is ready to listen and verification
    if ((listen(sockfd, 5)) != 0) {
        printf("Listen failed...\n");
        exit(0);
    }
    else
        printf("Server listening..\n");
    len = sizeof(cli);
   
    // Accept the data packet from client and verification
    connfd = accept(sockfd, (SA*)&cli, &len);
    if (connfd < 0) {
        printf("server accept failed...\n");
        exit(0);
    }
    else
        printf("server accept the client...\n");
   
    // Function for chatting between client and server
    func(connfd);
   
    // After chatting close the socket
    close(sockfd);
}

Client TCP :
C

#include <netdb.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#define MAX 80
#define PORT 8080
#define SA struct sockaddr
void func(int sockfd)
{
char buff[MAX];
int n;
for (;;) {
bzero(buff, sizeof(buff));
printf("Enter the string : ");
n = 0;
while ((buff[n++] = getchar()) != '\n')
;
write(sockfd, buff, sizeof(buff));
bzero(buff, sizeof(buff));
read(sockfd, buff, sizeof(buff));
printf("From Server : %s", buff);
if ((strncmp(buff, "exit", 4)) == 0) {
printf("Client Exit...\n");
break;
}
}
}

int main()
{
int sockfd, connfd;
struct sockaddr_in servaddr, cli;

    // socket create and verification
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1) {
        printf("socket creation failed...\n");
        exit(0);
    }
    else
        printf("Socket successfully created..\n");
    bzero(&servaddr, sizeof(servaddr));
   
    // assign IP, PORT
    servaddr.sin_family = AF_INET;
    servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    servaddr.sin_port = htons(PORT);
   
    // connect the client socket to server socket
    if (connect(sockfd, (SA*)&servaddr, sizeof(servaddr)) != 0) {
        printf("connection with the server failed...\n");
        exit(0);
    }
    else
        printf("connected to the server..\n");
   
    // function for chat
    func(sockfd);
   
    // close the socket
    close(sockfd);
}

Compilation –
Côté serveur :
gcc serveur.c -o serveur
./serveur

Côté client :
gcc client.c -o client
./client

Production –

Du côté serveur:

Socket successfully created..
Socket successfully binded..
Server listening..
server accept the client...
From client: hi
To client : hello
From client: exit
To client : exit
Server Exit...

Côté client:

Socket successfully created..
connected to the server..
Enter the string : hi
From Server : hello
Enter the string : exit
From Server : exit
Client Exit... 
