# SyncText:- CRDT based text file synchronization

# **Overview**

SyncText is a real-time collaborative text editor that allows multiple users to edit the same document simultaneously. Built using Conflict-Free Replicated Data Types (CRDT), it ensures conflict-free merging without locks, providing a Google Docs-like collaborative experience.

## **Features**

* **Real-time Collaboration**: Multiple users can edit simultaneously  
* **Automatic Change Detection**: Monitors file changes automatically  
* **Conflict-Free Merging**: Uses CRDT principles for consistency  
* **Lock-Free Operation**: No locks required for synchronization  
* **Multi-Threaded Architecture**: Concurrent file monitoring and message processing

## **Prerequisites**

### **System Requirements**

* **Operating System**: Linux (Ubuntu 20.04+ recommended)  
* **Compiler**: gcc/g++ 9.4.0 or later  
* **Libraries**: pthread, rt (for message queues)

### **Required Libraries**

bash  
sudo apt-get update  
sudo apt-get install build-essential

## **Compilation Instructions**

### **Step 1: Compile the Source Code**

bash  
gcc \-o editor synctext.c \-lrt \-lpthread

### **Flags Explanation:**

* `-lrt`: Links the real-time library for message queues  
* `-lpthread`: Links the pthread library for multi-threading

### **Step 2: Verify Compilation**

bash  
ls \-la editor

You should see the `editor` executable file.

## **Execution Instructions**

### **Step 1: Start Multiple User Instances**

Open separate terminal windows for each user:

**Terminal 1 (User 1):**

bash  
./editor user\_1

**Terminal 2 (User 2):**

bash  
./editor user\_2

**Terminal 3 (User 3):**

bash  
./editor user\_3

### **Step 2: Verify Registration**

Each terminal should display:

text  
\=== SyncText CRDT Editor \===  
User: user\_1 | Document: user\_1\_doc.txt  
Last updated: \[current time\]  
Active users: user\_1, user\_2, user\_3  
Local ops: 0/5 | Received ops: 0  
\----------------------------------------  
Line 0: Hello World  
Line 1: This is a collaborative editor  
Line 2: Welcome to SyncText  
Line 3: Edit this document and see real-time updates  
\----------------------------------------  
Monitoring for changes... (Ctrl+C to exit)

## **Testing the System**

### **Test Scenario 1: Non-Conflicting Edits**

1. **User 1 makes changes:**  
   bash

nano user\_1\_doc.txt

1. Change Line 0 from "Hello World" to "Hello Everyone"  
   Save and exit  
2. **Observe in Terminal 1:**  
   * Change detection message appears  
   * Local operations counter increases  
   * After 5 operations or timeout, changes broadcast to others  
3. **Check other terminals:**  
   * Received operations counter increases  
   * Documents automatically update after merge

### **Test Scenario 2: Conflicting Edits**

1. **User 1 and User 2 edit same line simultaneously:**  
   * User 1: Change Line 1 to "This is an AMAZING editor"  
   * User 2: Change Line 1 to "This is a GREAT editor"  
2. **Observe Conflict Resolution:**  
   * System detects conflict on same line  
   * Last-Writer-Wins strategy applied  
   * User with later timestamp wins  
   * All users converge to same content

### **Test Scenario 3: Multiple Users**

1. **Start 3-5 users as shown above**  
2. **Each user makes different edits:**  
   * User 1: Modify first line  
   * User 2: Modify second line  
   * User 3: Add new line  
3. **Verify all changes propagate correctly**

## **File Structure**

text  
.  
â”œâ”€â”€ editor          \# Compiled executable  
â”œâ”€â”€ synctext.c      \# Main source code  
â”œâ”€â”€ user\_1\_doc.txt  \# User 1's document (auto-created)  
â”œâ”€â”€ user\_2\_doc.txt  \# User 2's document (auto-created)  
â””â”€â”€ user\_3\_doc.txt  \# User 3's document (auto-created)

## **System Architecture**

### **Components Created Automatically:**

* **Shared Memory Registry**: `/synctext_registry`  
* **Message Queues**: `/queue_user_1`, `/queue_user_2`, etc.  
* **Semaphore**: `/registry_sem`  
* **User Documents**: `user_X_doc.txt`

### **Threads per User Instance:**

1. **Main Thread**: CRDT merging and display  
2. **Monitor Thread**: File change detection  
3. **Listener Thread**: Message reception  
4. **Heartbeat Thread**: Registry updates

## **Important Notes**

### **Performance Characteristics:**

* **Change Detection**: Polls every 2 seconds  
* **Heartbeat Updates**: Every 3 seconds  
* **Merge Trigger**: After 5 operations or when received ops exist  
* **Broadcast**: Every 3 seconds if local operations exist

### **Resource Limits:**

* **Maximum Users**: 5 concurrent users  
* **Maximum Lines**: 1000 lines per document  
* **Message Queue Size**: 10 messages per queue  
* **Operation Buffer**: 5 local \+ 25 received operations

### **Cleanup:**

The system automatically cleans up on exit (Ctrl+C):

* Unregisters user from shared memory  
* Closes and unlinks message queues  
* Releases all allocated memory

## **Troubleshooting**

### **Common Issues:**

1. **Compilation Error: "mq\_open: No such file or directory"**  
   bash

sudo apt-get install librt-dev

**Permission Denied for Shared Memory**

bash  
sudo sysctl \-w kernel.msgmnb=65536  
sudo sysctl \-w kernel.msgmax=65536

**Message Queue Limits**

bash  
\# Check current limits  
cat /proc/sys/fs/mqueue/msg\_max  
cat /proc/sys/fs/mqueue/msgsize\_max

**Cleanup Stale Resources**

bash  
\# Remove leftover message queues  
sudo rm \-rf /dev/mqueue/\*

\# Remove shared memory  
sudo rm \-rf /dev/shm/\*

### **Debug Mode:**

Add debug prints by uncommenting printf statements in the code to see detailed operation flow.

## **Example Output**

### **Successful Operation:**

text  
Registered as user\_1  
Message queue created: /queue\_user\_1  
Active users: user\_1, user\_2

Detected change: line 1, type 2, seq 1  
Broadcasted to 2 users  
Received: user\_2 line 1 type 2 seq 3

\=== CRDT MERGE STARTED \===  
Local ops: 1, Received ops: 1  
Total operations to merge: 2  
Applied 2/2 operations  
\=== CRDT MERGE COMPLETED \===

## **Support**

For issues with compilation or execution, ensure:

1. All prerequisites are installed  
2. File permissions are correct  
3. System resources are available  
4. No stale resources from previous runs

