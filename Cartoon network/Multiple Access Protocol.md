
There are three type of MAP:



1. Random Access Protocol : it has four type :  1.Aloha 2. CSMA 3. CSMA/CD 4. CSMA/CA
2. Controlled Access Protocol : it has three type : 1. Reservation 2. polling 3. token passing
3. Channelization protocol : it has three type : 1. FDMA 2. CDMA 3. TDMA


---------------------------<<<<<<<<<<<>>>>>>>>>>>>>--------------------



### **ALOHA Protocol**

ALOHA is a simple communication protocol used in networks to manage how devices send data over a shared channel. It works on the principle of **"send and hope"**, meaning devices send data whenever they want, and if a collision (two devices sending at the same time) occurs, they wait and resend after some time.

#### **Types of ALOHA**

1. **Pure ALOHA:**
    
    - Devices send data **anytime** without checking if the channel is free.
    - If a collision happens, the device waits a random amount of time before resending.
    - Efficiency is low because collisions can occur frequently.
2. **Slotted ALOHA:**
    
    - Time is divided into fixed slots.
    - Devices can only send data at the **start of a time slot**.
    - Reduces the chances of collision compared to Pure ALOHA, resulting in better efficiency.

---

### **CSMA (Carrier Sense Multiple Access)**

CSMA is a protocol used to avoid collisions in a shared channel. It works on the principle of **"listen before talk"**, meaning devices check whether the channel is free before sending data.

#### **Types of CSMA**

1. **1-Persistent CSMA:**
    
    - A device continuously listens to the channel.
    - If the channel is free, it sends data immediately.
    - If the channel is busy, it waits until it's free and sends data instantly, which may still cause collisions.
2. **Non-Persistent CSMA:**
    
    - A device listens to the channel.
    - If the channel is busy, it waits for a random amount of time before checking again.
    - Reduces collision chances but increases delay.
3. **P-Persistent CSMA:**
    
    - Used in slotted channels.
    - When the channel is free, the device sends data with a probability **P**.
    - If not sent, it waits for the next time slot and repeats the process.
4. **CSMA/CD (Collision Detection):**
    
    - Used in Ethernet.
    - Devices detect collisions while transmitting data.
    - If a collision occurs, they stop transmission and wait for a random time before retrying.
5. **CSMA/CA (Collision Avoidance):**
    
    - Used in Wi-Fi networks.
    - Devices use techniques to **avoid collisions** by reserving the channel before sending data.
    - They send a signal (e.g., RTS/CTS) to ensure no other device uses the channel.

In simple terms, **ALOHA is simpler but less efficient**, while **CSMA tries to minimize collisions** by sensing the channel before sending data.


---------------------------<<<<<<<<<<<>>>>>>>>>>>>>--------------------


### **Controlled Access Protocols**

Controlled Access Protocols ensure that devices share the network channel without collisions by controlling access to the channel. Only one device transmits at a time, following strict rules.

---

#### **1. Reservation Protocol**

- **How it works:**
    - Time on the channel is divided into slots.
    - Devices reserve slots before transmitting data.
    - A special control message is sent to make the reservation.

---

#### **2. Polling Protocol**

- **How it works:**
    - A central controller (or master device) polls each device to check if it has data to send.
    - Only the polled device is allowed to transmit.
    - If no data is available, the master moves to the next device.

---

#### **3. Token Passing Protocol**

- **How it works:**
    - A special control frame, called a **token**, is passed between devices in a logical order.
    - A device can only transmit data when it has the token.
    - After transmitting, the device passes the token to the next device.


---------------------------<<<<<<<<<<<>>>>>>>>>>>>>--------------------