Download Link: https://assignmentchef.com/product/solved-cs39006-assignment-7-reliable-communication-over-unreliable-links
<br>
<strong>Objective:  </strong>

The objective of this assignment is to build support for reliable communication over an unreliable link. The unreliable link will be implemented with a UDP socket. You will have to give APIs to the user that will do reliable send/receive over this unreliable link by appropriate implementation of the APIs.

<strong>Problem Statement:</strong>

In this assignment, you will be building support for reliable communication over an unreliable link. In particular, you will build a message-oriented, reliable, exactly-once delivery communication layer (you can think of it as TCP without byte-orientation and in-order delivery, or UDP with reliable delivery). Message ordering is not needed, so messages with higher ids can be delivered earlier.




You have been introduced to the function calls <em>socket</em>, <em>bind</em>, <em>sendto</em>, and <em>recvfrom</em> to work with UDP sockets. Assume that the TCP sockets are not there. We know that UDP sockets are not reliable, meaning that there is no guarantee that a message sent using a UDP socket will reach the receiver. We want to implement our own socket type, called

MRP (<em>My Reliable Protocol</em>) socket, that will guarantee that any message sent using a

MRP socket is always delivered to the receiver exactly once. However, unlike TCP sockets, MRP sockets may not guarantee in-order delivery of messages. Thus messages may be delivered out of order (later message delivered earlier). However, a message should be delivered to the user exactly once.




To implement each MRP socket, we use the following:




<ol>

 <li>One UDP socket through which all actual communication happen.</li>

 <li>One thread X. Thread X handles all messages received from the UDP socket, and all timeout and retransmissions needed for ensuring reliability.</li>

 <li>At least the following three data structures associated with it (you can have others of your own if you need them): <strong><em>receive buffer</em></strong>, <strong><em>unacknowledged-message table</em></strong>, and <strong><em>received-message-id table</em></strong>. The <strong><em>receive buffer</em></strong> contains the application messages that are received. This will contain each message sent exactly once before it is read by the user. The <strong><em>unacknowledged-message table</em></strong> contains the details of all messages that have been sent but not yet acknowledged by the receiver, along with the last sending time of the message. This will be used to decide which messages need to be retransmitted if no acknowledgement is received. The <strong><em>received-message-id table</em></strong> contains all distinct message ids that are received in the socket so far. This will be used to detect duplicate messages.</li>

</ol>

You can assume a maximum size of 100 bytes for each message and maximum table sizes of 100 messages for each table. The tables should be <strong>dynamically created when the socket is opened</strong> and <strong>freed when the socket is closed</strong>. The table pointers (or pointers to any other data structure you may need) can be kept as global variables, so that they will be accessible from all API functions directly.




The broad flow on send and receive of messages is as follows:




<strong><em>Sending an application message for the first time:</em></strong> The user calls an API function <em>r_sendto()</em> to send its message. The send API adds a unique id (use a counter starting from 0) to the application message and sends the message with the id using the UDP socket (this should happen in a single sendto() call). It also stores the message, its id, destination IP-port, and the time it is sent in <strong><em>unacknowledged-message table.</em></strong> This is done in the user thread (main process) and the thread X  has no role here. The send API function returns after the send to the user.




<strong><em>Handling message receives (application and ACK) from the socket and Retransmission of Messages:</em></strong> The thread X waits on a select() call to receive message from the UDP socket or a timeout T. Given that X can come out of the wait either on receiving  a message or on a timeout, its actions on each case are as follows:

<ul>

 <li>If it comes out of wait on a timeout, it calls a function <strong><em>HandleRetransmit()</em></strong>. This function handles any <strong><em>retransmission of application message</em></strong> The function scans the <strong><em>unacknowledged-message table</em></strong> to see if any of the messages’ timeout period (T) is over (from the difference between the time in the table entry for a message and the current time). If yes, it retransmits that message and resets the time in that entry of the table to the new sending time. If not, no action is taken. This is repeated for all messages in the table. At the end of scanning all messages, the function returns. On return of the function, the thread X goes to wait on the select() call again with a fresh timeout of T.</li>

 <li>If it comes out of wait on receiving a message in the UDP socket, it calls a</li>

</ul>

function <strong><em>HandleReceive()</em></strong>. The function checks if it is an application message or an ACK.

<ul>

 <li>If it is an application message, call a function <strong><em>HandleAppMsgRecv()</em></strong>, which does the following: o check the <strong><em>received-message-id table</em></strong> if the id has already been received (duplicate message). If it is a duplicate message, the message is dropped, but an ACK is still sent. If it is not a duplicate message, the message is added to the <strong><em>receive buffer</em></strong> (without the id) including source IP-port and an ACK is sent. The function returns after that.</li>

 <li>If it is an ACK message, call a function <strong><em>HandleACKMsgRecv()</em></strong>, which does the following:</li>

</ul>

o If the message is found in the <strong><em>unacknowledged-message table</em></strong>, it is removed from the table. If not (duplicate ACK), it is ignored. The function returns after that.

After handling the message, the HandleReceive() function returns. On return of the function, the thread X goes to wait on the select() call again <strong><em>with a timeout of T</em></strong><strong><em><sub>rem</sub></em></strong><strong><em><sup>, where T</sup></em></strong><strong><em><sub>rem</sub> ≤ T is the timeout remaining when the select() call came out of wait because of the message receive</em></strong>.




<strong><em>Receiving a message by the user:</em></strong> The user calls an API function <em>r_recvfrom()</em>, which either finds a message in the receive buffer or not. If there is a message in the receive buffer, the first message is removed and given to the user, and the function returns the no. of bytes in the message returned. If there is no message in the receive buffer, the user process is blocked. So there is again no direct role of the thread X here. The function will return when there is a message in the receive buffer (if a message comes, it will be put in the buffer by X in the background).




You can assume there will be exactly one MRP socket created by a process for send and receive, so only one set of the variables/threads are needed.




You will be implementing an API with a  set of function calls <em>r_socket</em>, <em>r_bind</em>, <em>r_sendto</em>, <em>r_recvfrom</em> , and <em>r_close </em>that implement MRP sockets. <strong>The parameters to these functions and their return values are exactly the same as the corresponding functions of the UDP socket, <em>except for r_socket</em></strong>. The functions will be implemented as a static library. Any user wishing to use MRP sockets will write a C/C++ program that will call these functions in the same sequence as when using UDP sockets. The library will be linked with the user program during compilation.




A brief description of the functions is given below. All calls should return 0 on success (except r_recvfrom() which should return the no. of bytes in the message) and -1 on error.




<ul>

 <li><em>r_socket</em> – This function opens an UDP socket with the <em>socket</em> It also creates the thread X, dynamically allocates space for all the tables, and initializes them. The parameters to these are the same as the normal socket( ) call, except that it will take only SOCK_MRP as the socket type.</li>

 <li><em>r_bind</em> – Binds the UDP socket with the specified address-port using the <em>bind</em></li>

 <li><em>r_sendto</em> –Adds a message id at the beginning of the message and sends the message immediately by <em>sendto</em>. It also stores the message along with its id and destination address-port in the <strong><em>unacknowledged-message table</em></strong> before sending the message. With each entry, there is also a time field that is filled up initially with the time of first sending the message.</li>

 <li><em>r_recvfrom</em> – Looks up the <strong><em>received-message table</em></strong> to see if any message is already received in the underlying UDP socket or not. If yes, it returns the first message and deletes that message from the table. If not, it blocks the call. To block the <em>r_recvfrom</em> call, you can use <em>sleep</em> call to wait for some time and then see again if a message is received. <em>r_recvfrom</em>, similar to <em>recvfrom</em>, is a blocking call by default and returns to the user only when a message is available.</li>

 <li><em>r_close</em> – closes the socket; kills all threads and frees all memory associated with the socket. If any data is there in the <strong><em>received-message table</em></strong>, it is discarded.</li>

</ul>




Design the message formats and the <strong><em>unacknowledged-message table</em></strong> and the <strong><em>receivedmessage tables</em></strong> properly. Note that the tables are sometimes shared between different threads and would require proper mutual exclusion.







<strong>Testing your code: </strong>




<ol>

 <li>Write a file <strong><em>h</em></strong> that will contain

  <ol>

   <li>#includes for the sockets to work</li>

   <li>#define for the timeout T (set to 2 seconds) and the drop probability <em>p</em> (see description of <strong><em>dropMessage()</em></strong> function below)</li>

   <li>prototypes of all the functions in the API listed above + a function dropMessage() which is described below.</li>

  </ol></li>

 <li>Write a file <strong><em>c</em></strong> that contain all global variable declarations for the thread X and the data structures, and implementation of all the functions in the API + <strong><em>dropMessage()</em></strong>. This should NOT contain any main() function.</li>

 <li>Create a static library called <strong><em>a</em></strong> (look up the <strong><em>ar</em></strong> command in linux)</li>

 <li>Write two test programs <em>c</em> and <em>user2.c</em>. The program in <em>user1</em>.c will create a MRP socket M1 and bind it to the port 50000+2*&lt;your roll no&gt; (for ex., if your roll no. is 1001, the port no. is 52002). It then reads a long string from the keyboard (25 &lt; string size &lt; 100), and sends each character of the string <strong>in a separate message</strong> to<em> user2.c</em>. The messages are sent using M1 by making the <em>r</em>_<em>sendto</em> calls. The program in <em>user2.c</em> will create a MRP socket M2 and bind it to the port 50000+2*&lt;your roll no&gt; + 1 (for ex., if your roll no. is 1001, the port no. is 52002 + 1 = 52003). It then receives each message from <em>user2.c</em> using the <em>r_recvfrom</em> call on M2 and immediately prints the character received on the screen. Insert the <strong><em>dropMessage()</em></strong> function at appropriate place (see below).</li>

 <li>#include <em>h</em> in user1.c and user2.c and compile them with <em>librsocket.a</em> (For example, when we want to use functions in the math library like <em>sqrt()</em>, we include <em>math.h</em> in our C file, and then link with the math library <em>libm.a</em> byusing the flag – lm during compilation)</li>

 <li>Run <em>user1</em> and <em>user2 </em>to see if the strings are transmitted correctly. Also measure the metric below by inserting appropriate code in your library.</li>

</ol>




<strong><em><u>dropMessage()</u></em><u> function</u>:</strong>




As the actual number of drops in the lab environment will be near 0, you will need to simulate an unreliable link. To do this, in the library, add a function called <em> <strong>dropMessage()</strong></em> with the following prototype in your library:

<strong><em>int dropMessage(float p) </em></strong>




where <em>p</em> is a probability between 0 and 1. This function first generates a random number between 0 and 1. If the generated number is &lt; <em>p</em>, then the function returns 1, else it returns 0. Now, in the code for thread X, after a message is received (by the <em>recv_from()</em> call on the UDP socket), first make a call to <em>dropMessage()</em>. If it returns 1, do not take any action on the message (irrespective of whether it is data or ack) and just return to wait to receive the next message. If it returns 0, continue with the normal processing in X. Thus, if <em>dropMessage()</em> returns 1, the message received is not processed and hence can be thought about as lost. <strong>Submit your code with the <em>dropMessage()</em> calls in X, do NOT remove these calls from your code before you submit.</strong>




The value of <em>T</em> should be 2 seconds (#define in <em>rsocket.h</em>). The value of the parameter <em>p</em> (the probability) should also be specified in the <em>rsocket.h</em> file with a #define. When you test your program, vary <em>p</em> from 0.05 to 0.5 in steps of 0.05 (0.05, 0.1, 0.15, 0.2….,0.45, 0.5). For each <em>p</em> value, for the same string, count the average number of transmissions made to send each character (<em>total number of transmissions that are made to send the string /no. of characters in the string</em>). You can do this by adding a counter in the appropriate part of the code for thread X. Report this in a table in the beginning of the file <em>documentation.txt</em> (see below).







<strong>What to submit: </strong>




You should submit the following files in a single zip file:

<ol>

 <li><em>h</em></li>

 <li><em>c</em></li>

 <li><em>makefile</em> (this should include all commands to create a library named <em>a</em> from your source files.)</li>

 <li>A file called <em>txt</em> containing description of the different files. This file should include:

  <ol>

   <li>The table described above for the no. of retransmissions</li>

   <li>A list of all messages and their formats, with a brief description of the use of each field</li>

   <li>A list of all data structures used and a brief description of their fields</li>

  </ol></li>

</ol>

<strong> </strong>

<strong><u>Do not include any other file in your submission, and upload the submission only in</u> <u>zip format.</u>  </strong>