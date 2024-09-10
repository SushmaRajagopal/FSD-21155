
# Case Study 6: Real-Time Voting Application

---

## Run Locally

### Clone the project

```bash
git clone <repository-url>
```

### Go to the project directory

```bash
cd <directory-name>
```

### Install dependencies

```bash
yarn install
```

### Start the server

```bash
yarn run dev
```

---

## Questions

### 1. How would you implement a real-time WebSocket connection to handle voting updates?

To implement a real-time WebSocket connection for voting updates, I would use **Socket.IO**. Here’s how:

1. **Set Up a Node.js Server**:
   - Create a server using Express and initialize Socket.IO.

   ```javascript
   const express = require('express');
   const { createServer } = require('http');
   const { Server } = require('socket.io');

   const app = express();
   const server = createServer(app);
   const io = new Server(server);

   io.on('connection', (socket) => {
     console.log('A user connected');

     socket.on('vote', (option) => {
       io.emit('newVote', option); // Broadcast the vote to all clients
     });

     socket.on('disconnect', () => {
       console.log('A user disconnected');
     });
   });

   server.listen(3001, () => {
     console.log('Server is running on port 3001');
   });
   ```

2. **Client-Side Connection**:
   - In your React component, establish a WebSocket connection using Socket.IO.

   ```javascript
   import React, { useState, useEffect } from 'react';
   import io from 'socket.io-client';

   const socket = io('http://localhost:3001');

   const Voting = () => {
     const [votes, setVotes] = useState({ optionA: 0, optionB: 0 });

     useEffect(() => {
       socket.on('newVote', (option) => {
         setVotes((prevVotes) => ({
           ...prevVotes,
           [option]: prevVotes[option] + 1
         }));
       });
     }, []);

     return (
       <div>
         <h1>Voting</h1>
         <p>Option A: {votes.optionA}</p>
         <p>Option B: {votes.optionB}</p>
       </div>
     );
   };

   export default Voting;
   ```

### 2. How would you scale the application to handle thousands of users voting simultaneously?

To scale this voting application, I would:

- **Use Horizontal Scaling**: Deploy multiple instances of the server and use a load balancer to distribute incoming traffic across them.
  
- **Database Optimization**: Ensure the database can handle large volumes of writes by optimizing queries and using connection pooling.
  
- **Redis or Message Queues**: Use Redis or a message queue like RabbitMQ to manage vote tallies in real time, improving performance and reducing latency.
  
- **WebSocket Load Balancing**: Use WebSocket-aware load balancers like Nginx or AWS Elastic Load Balancer to distribute WebSocket connections across multiple servers.
  
- **Use a Distributed Event System**: For massive scale, consider systems like Kafka or Pub/Sub to handle and process votes in real-time.

### 3. How would you store and manage votes in the database to ensure efficiency and prevent double voting?

To efficiently manage and store votes, I would:

- **Schema Design**: Use a simple relational schema with tables like `users`, `topics`, and `votes` with foreign keys to track who voted for what.
  
- **Unique Constraints**: Apply a unique constraint on the combination of user ID and topic ID in the `votes` table to prevent duplicate voting.
  
- **Transactions**: Use database transactions to ensure votes are consistently written, preventing partial or invalid records.
  
- **Indexes**: Add indexes on relevant columns like user ID and topic ID to speed up query performance during voting.

### 4. Explain how you would handle user authentication and restrict users to one vote per topic.

To handle user authentication and restrict one vote per user:

1. **User Authentication**:
   - Use JWT (JSON Web Tokens) or OAuth for authentication.
   - Securely store the user’s session in a cookie or local storage.
   - Require users to log in before they can vote.

2. **Vote Restriction**:
   - Store votes in a database with unique identifiers (user ID + topic ID).
   - Before allowing a vote, check if the user has already voted on that topic. If so, block additional votes.
   - Enforce this rule on both the client and server sides for added security.

### 5. What strategies would you use to ensure the reliability and accuracy of the voting results?

To ensure voting reliability and accuracy, I would:

1. **Data Validation**:
   - Validate vote data on both client and server sides to prevent invalid data from being processed.

2. **Database Integrity**:
   - Use transactions to ensure that votes are recorded correctly and to prevent partial writes.
   - Apply unique constraints to avoid double voting for the same user and topic.

3. **Audit Logs**:
   - Maintain audit logs of voting actions, including user ID, topic ID, and timestamps, to track voting events and resolve discrepancies.

4. **Concurrency Handling**:
   - Use optimistic locking or similar mechanisms to handle concurrent voting attempts and ensure vote counts are correct.

5. **Redundancy**:
   - Store vote data in multiple locations (e.g., replicate databases) to avoid data loss in case of failures.
   - Use distributed systems like Kafka to process votes in real-time for scalability and fault tolerance.

6. **Monitoring and Alerts**:
   - Set up monitoring tools to track vote processing and set alerts for any unusual behavior or errors.

---

