# Distributed Systems Project - Parallel and Distributed Computing

The work to which this document refers, within the scope of the Parallel and Distributed Computing course, aims to create a client-server system using TCP sockets in Java that allows multiple users to play the game "Who gets closest". To achieve this, the developed solution allows players to authenticate and then join one of two lobbies (Simple Lobby and Rank Lobby), which will constitute matches with a configurable number of players, in different ways. Once the game group is formed, a new instance of Game is created, starting a match between the selected players.


## O Jogo
The game implemented consists of elimination rounds. In each round, a random integer from 0 to 100 is generated. Each player is challenged to guess the number in play.

At the end of the round, the player whose guess is furthest from the correct answer is eliminated, causing the game to proceed to the next round with one less competitor, and so on.

Naturally, the last round will be played between two players, with the one whose guess is closest to the actual number in that round emerging as the winner.
Uma partida do jogo implementado é composta por rondas eliminatórias. Em cada ronda, é gerado um número inteiro aleatório de 0 a 100. Cada jogador é desafiado a adivinhar qual o número em jogo, dando o seu palpite.


At the end of the round, the player whose guess is furthest from the correct answer is eliminated, causing the game to proceed to the next round with one less competitor, and so on.

Naturally, the last round will be played between two players, with the one whose guess is closest to the actual number in that round emerging as the winner.

## Compilation and Execution

Before performing any other action, you should execute the `make` command to compile the files.

Then, to run the server, execute the `make run_server` command.

It is also possible to start the server by specifying the port where it will listen and the number of players required to start a game using `make run_server PORT=<PORT> NUM_PLAYERS=<NUM_PLAYERS>`.

To run the client, execute the `make run_client` command.

It is also possible to run the client by customizing the host and port where it will look for the server using `make run_client HOST=<HOST> PORT=<PORT>`.


## Server

### Architecture

#### Database

A `csv` file that ensures the persistence of registered player information, including username, password, and total score.

<br>

#### Server.java:

This class represents the server. Initially, there is only one thread responsible for accepting new connections. Whenever a new connection is established (new socket), a new thread is created to handle all messages from the corresponding socket.

The following are the messages that the server expects to receive from clients, characterized by their syntax and the action they trigger:

- `AUTH <username> <password>`

    - Action: Authenticates the player with the provided username and password. If successful, it sends a message with the token assigned to the client, which should be used in subsequent messages to identify itself. If the server detects that the player was in a lobby or a game, it sends that information to the client so that it can restore its corresponding internal state.

- `REGISTER <username> <password>`

    - Action: Registers a new player with the provided username and password.

- `SIMPLE <token>`

    - Action: Adds the player to the simple lobby if the player is not in a lobby or a game.

- `RANK <token>`

    - Action: Adds the player to the rank lobby if the player is not in a lobby or a game.

- `LEAVE_LOBBY <token>`

    - Action: Removes the player from the lobby if the player is in one.

- `POINTS <token>`

    - Action: Returns the current number of points the player has.

- `PLAY <token> <guess>`

    - Action: If the player is in a game, sets the indicated value as their guess for the current round.

If the server receives a message that is not listed above, it will respond with `ERROR: Command: Invalid command.`.


#### Player.java

This class represents a player, **from the server's point of view**.

Additionally, the class statically stores the set of all players who currently have an active session. 

Each object stores the player's username, their number of points, their current token, and the last socket through which the player communicated with the server. The password is only stored in the database.
Although the client's corresponding socket is stored in each object, the player can send messages through any socket (as long as they are properly identified with the respective token). If it is different from the stored socket, it will be updated.

All messages sent to the player will be sent through the last known socket.

#### SimpleLobby.java

This class is the implementation of the simple lobby. 
Any player can join this lobby. Once the required number of players to start a game is reached, a game is initiated with those players, regardless of their score, and the lobby is emptied.

#### RankLobby.java

Here, the rank lobby is implemented. 

This lobby tries to create games with players whose total scores are similar, but this condition is gradually relaxed so that players don't have to wait indefinitely for a game.

Any player can join this lobby. Upon joining, they will be assigned a search radius, which dictates the range of scores of players they can start playing with, initialized at 0 points and incremented by 5 units every second.

Additionally, every second, the RankLobby creates all sets where part of the search radius is contained within all other elements of these sets.

If any of these sets contains the required number of players, a game is initiated with them, and the players are removed from the RankLobby.

The process is executed by a dedicated thread for the Rank Lobby.

To create the sets, the program sorts all input and output values (for example, a player with a score of 2000 points and a search radius of 300 at the moment will have an input value of 2300 and an output value of 1700). Then, the sorted list is iterated, and whenever an input value appears, the corresponding player is added to the list of open intervals; whenever an output value appears, the corresponding player is removed from the list of open intervals, and a copy of this set is added to the list of sets whose search radii are contained within all other intervals in the group.


#### Game.java

Responsible for implementing a game match and the interaction throughout its rounds.

<br><br><br><br>

## Client

### Architecture

#### Client.java

The Client class is the main entry point of the client-side. It maintains an internal state influenced by user input and messages received from the server. It is responsible for providing the text-based user interface. Additionally, it stores the token provided by the server, which is essential for identifying itself in the messages it sends.

#### ClientState.java

Essential for the internal representation of a client's state. It specifies the possible states and the transitions between them based on messages received from the server.

#### ClientStub.java

Acts as the entry and exit point for communication with the server. It establishes and closes the communication socket and ensures the sending and receiving of messages to and from the server associated with the socket, based on the specified hostname and port.

### Execution Example

Upon starting the application, the user will see the authentication menu with options for authentication, registration, and exit. Depending on the choice, the application will prompt for the necessary information and communicate with the server, which will process it, trigger the corresponding action, and return a response.

After authentication, the user can choose between different game lobbies.

During a match, the user will be prompted to make their guess (effectively, execute their move) and will be presented with information about the progress of the game, including their own performance as well as that of the opponents in each round.

## SSL

All communications between the server and client are carried out through sockets that adhere to the Secure Sockets Layer (SSL) protocol, using the SSLSocket library, to ensure the security of all messages, especially those related to player authentication and registration.

For this purpose, a certificate (`src/server/certificate/keystore.jks`) has been created, which is necessary for a client to establish a communication socket with the server.

## Note

The key press detection mechanism present in the Keyboard class was adapted from Stack Overflow (https://stackoverflow.com/questions/18037576/how-do-i-check-if-the-user-is-pressing-a-key) due to its complexity.

***

- Antnio Augusto de Sousa
- Pedro de Almeida Lima
- Pedro Simão Januário Vieira

