## Project 4:  Reliable Transport Protocol
### Description  
This is a design for a simple transport protocol that provides reliable datagram service. The protocol will be responsible for ensuring data is delivered in order, without duplicates, missing data, or errors.
The code will transfer a file reliably between two nodes (a sender and a receiver). Assuming that the receiver runs first and will wait indefinitely, the sender can just send the data to the receiver.
 
### Approach  
We started with the code provided by the professors, which implements both sender and receiver files with their corresponding classes and methods. These include the _log_, _send_, and _run_ that print out to STDERR, send a message through the socket, and run the program respectively. 

#### Receiver
The _run_ method in the _3700recv_ program, checks if the received message can be loaded as a JSON and then sees if the message received corresponds to the one sent by the sender with the use of the _checksum_ method. If the previous two conditions are met we check if the message received is in order and if it is we print it out, otherwise we add it to a queue of messages sent in the wrong order. After checking if it is in order we go through this queue checking if any of those can be printed because they correspond with the right order and do so. After this process we always send an acknowledgement back, that contains the last sequence number received in the right order.

To check if both sent and received messages correspond we perform checksum, checking if the message sent and received have a complementary representation in binary, meaning the sum of both is all ones.

#### Sender
3700send also contains a _run_ method with a loop where we manage both the ACKs received and the sending of the data. First of all, we use _select.select()_ to check if there are bytes to read in the sockets. 

- If there are bytes to read: we read the ACK, compute the checksum and if it is correct we check if it is a duplicate ACK or not. If the ACK is neither duplicated nor corrupted then we update the congestion window (following the slow start and congestion avoidance rules), the last sequence number acknowledged, the space available on the window and the RTT. To update the RTT we make the following weighted sum: 0.2*new_RTT + 0.8*old_RTT.
Finally, if there are no ACKs pending to be received then we stop waiting for them (self.waiting = False) so we can start sending the next data block.

- If there are bytes to send (i.e. self.waiting == False), while the window is big enough to send packets, we read the data from STDIN, in case there is no more data, we exit the loop to send data, otherwise we send the data.

If timeout has occurred, it is managed at the beginning of the while loop. There we check if the last ACK received is lower than the last sequence number sent. If that’s the case, and the time that has passed is two times the RTT, then we resend all the message from the last acknowledged (not included) to the last sended. We also update ssthresh and window accordingly, following the protocol of TCP Tahoe.

Finally, the sender exits when all the data was sent and there are no ACKs left to receive.
 
### Challenges and difficulties
The hardest part of this project was debugging. Since we are using a server it is really difficult to see what we are doing at all times as the only way is by printing everything in the console. The implementation for the level 7 tests was the worst since it messed up our previous work. However, I think this project has helped us get more comfortable with and have a better understanding of the TCP protocol.  

 
### Testing and debugging
To do the testing we used all the tests provided in the configs folder. For debugging the _log_ function was pretty useful.

Until test 7 everything was working pretty fine. We were obtaining the expected results based on the changes made. However, after implementing the congestion window we started to experience some weird behaviors in the previous test, especially in tests 5.1, 6.3, and 7.1. We observed that the RTT value needed to be lowered to detect packet drops efficiently (we were never reaching the timeout to resend).

Then, we decided to try a totally different approach: instead of doing the ACK of the corresponding sequence number, we started doing the ACK of the last sequence number received in the right order. This was the first step to implementing TCP Reno, but we experienced a much better performance just by adapting the whole code to this change. Then, our TCP Reno implementation was not as good as the previous approach, so we kept the good one.

We really wanted to end our TCP Reno implementation, but we did not have the time to do it.

