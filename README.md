To demonstrate my network:

![alt text](https://github.com/NineTheGreenHand/Blockchain-Socket-Programming/blob/main/Demonstration%20of%20Network.png?raw=true)

What is this project about:

    I wrote the serverA.cpp, serverB.cpp, serverC.cpp (three backend servers); 
    serverM.cpp (main server); clientA.cpp, clientB.cpp (two clients)
    The clientA and clientB connect with the main server with TCP connection, and
    the main server connets with backend servers with UDP connection.
    I designed the message format for the messages transfers between the client 
    and main server, and the main server to the backend servers.

What are my code files, and what each one of the does:
    
    -- clientA.cpp and clientB.cpp:

        Both are my client files, they connect to serverM with TCP connection,
        and client will have three operations: 1. check wallet 2. transfer coins
        3. TXLIST (ask for serverM to generate "alichain.txt", a sorted log file
        based on seq number). My client files will generate messages sent to the
        main server based on the operation they wish to perform. After they send
        the message, they print on-screen messages based on the operation. After 
        getting result back from the main server, my client will analyze the result 
        and generate on-screen messages based on the results. Terminates by themself
        once the operation asked is done.

    -- serverM.cpp:

        The main server. The main server is responsible for dealing with the command
        sent by the client. Main server analyze the command, and it connects the three
        backend servers (serverA, serverB, and serverC) with UDP connection. After 
        receive message from the client, it prints on-screen message. After analyzing
        the command, it sends the query to the backend servers to gather information 
        related to the client's command. Before sending the query to and after getting
        result from the backend servers, main server prints proper on-screen messages.
        After receiving all information from the backend servers, it analyzes the result,
        generates the message, and sends the message back to client. For the transfer 
        coins and TXLIST command, main server also responsible to generate the new log 
        entry and randomly select a backend server to add such log entry to its backend 
        log file, and to generate the "alichain.txt" file. Will not terminate by itself
        unless manually terminate with "Ctrl + C".

    -- serverA.cpp, serverB.cpp, serverC.cpp:

        Three backend servers. They connect with the main server with UDP connection.
        They analyze the message received from the main server, print on-screen messages
        after they receive the message from the main server, and after they send the
        message to the main server. Based on the message received, they gather asked 
        information and send it to the main server. For the transfer coins operation, 
        if the backend server is chosen for writing the new log, it will write the new
        log entry to its log file. For the TXLIST operation, it is responsible to send
        all the log entries back to the main server. Will not terminate by itself unless
        manually terminate with "Ctrl + C".

Format of all the messages exchanged: (I have a lot of comments made about this in the code files)

    -- Between clientA/B and serverM:

        1. clientA/B sends to serverM message format:

            (1) "cw username": cw stands for check wallet, username is the user balance to check

            (2) "txc sender receiver #coins": txc stands for transfer coins, sender is the sender
                                              of this transaction, likewise for the receiver, 
                                              #coins is the number of Alicoins wish to transfer

            (3) "txl": the TXLIST command

        2. serverM sends to clientA/B message format:

            (1) For the check wallet operation:

                (i) "cw s balance": cw stands for check wallet, s stands for success (user found),
                                    balance is the balance of the user
                
                (ii) "cw f1": cw stand for check wallet, f1 implies user not in the network (fail 1)

            (2) For the transfer coins operation:

                (i) "txc s balance": txc stands for transfer coins, s stands for success (transaction
                                     made successfully), balance is the sender's balance AFTER the
                                     transaction is made

                (ii) "txc f1 balance": f1 stands for (fail 1) means there is not enough balance for
                                      the sender to make the transaction, balance is the sender's
                                      current balance
                
                (iii) "txc f2s": f2s stands for the sender is not in the network

                (iv) "txc f2r": f2r stands for the receiver is not in the network

                (v) "txc f3": f3 stands for both the sender and the receiver not in the network

            (3) There is no message to send back to the client for TXLIST operation.

    -- Between serverM and serverA/B/C:

        1. serverM sends to serverA/B/C:

            (1) "cw username": cw stands for check wallet, username is the user balance to check

            (2) "txc sender receiver #coins": txc stands for transfer coins, sender is the sender
                                              of this transaction, likewise for the receiver, 
                                              #coins is the number of Alicoins wish to transfer

            (3) "txl": the TXLIST command

            (4) "wlog seq sender receiver #coins": wlog stands for write new log entry command, 
                                                   seq is the seq number of the new log entry,
                                                   sender, receiver, and #coins are information
                                                   in the new log entry

        2. serverA/B/C sends to serverM:

            (1) For the check wallet operation:

                (i) "found <transaction summation>": found implies that the user is found in the 
                                                     log file, transaction summation is the summation
                                                     of the coins received and sent. For example, 
                                                     if user received 100 coins, and sent 50 coins,
                                                     this value would be 100 + (-50) = 50
                
                (ii) "notfound": implies the user is not found in the log file.

            (2) For the transfer coins operation:

                (i) "found found maxSeq <transaction summation>": first found implies sender is found
                                                                  in the log file, second found implies
                                                                  receiver is found in the log file, 
                                                                  maxSeq is the max seq number in the 
                                                                  log file, <transaction summation>
                                                                  is the sender's coin summation of 
                                                                  coins received and sent.

                (ii) "found notfound maxSeq <transaction summation>": found sender, not found receiver
                
                (iii) "notfound found maxSeq 0": not found sender, found receiver, 0 is used since sender
                                                 is not found

                (iv) "notfound notfound maxSeq 0": not found sender and the receiver


            (3) For the TXLIST operation:

                "numLogs\nlog#1\nlog#2\nlog#3.......": numLogs stands for the number of logs in that log 
                                                       file, for log entry #1, #2, #3,......, separate 
                                                       them with "\n". So later can parse this message 
                                                       by "\n"

Idiosyncrasy of my project:

    Please start the process with this order: serverX, serverY, serverZ, serverW, clientA, clientB
    (X, Y, Z, W could be A, B, C, M, order does not matter). Start the servers first, or the program
    will not operate properly. Also, please all clientA and clientB in order. For example, always start
    with clientA, then clientB, then clientA, then clientB... The program will not work if this order is
    not followed. For example, if you start clientB first, it will not work. If you do two clientA or clientB
    in a row, it will not work. 
