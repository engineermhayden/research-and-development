To build a self-hosted, open-source alternative to a Discord server, with the use case you're describing (real-time alerting system), a few good alternatives can be considered. I'll outline the best options for you, keeping in mind the need to run them in Docker and integrate with Django REST Framework (DRF) microservices for alerting.

### 1. **Mattermost**
**Mattermost** is an open-source, self-hosted messaging platform that is a great alternative to Slack or Discord. It supports direct messages, group messages, and channels. It's designed for collaboration, but it could work well for real-time alerting and message broadcasting.

- **Docker Support**: Mattermost offers official Docker images, so it's easy to set up in a containerized environment.
- **Real-time Messaging**: It supports WebSocket-based real-time communication, which would fit your need for displaying messages in real-time on a wall-mounted monitor.
- **Extensibility**: Mattermost has a REST API that you can use to send messages from your Django services to the platform. You can also customize Mattermost with plugins and integrations if needed.
- **Features**:
  - Channels, threads, and private messaging.
  - Rich formatting for messages, including links, images, and code snippets.
  - File sharing and searching.
  - WebSocket-based real-time messaging.

**Setup with Docker:**
You can deploy Mattermost using Docker Compose. The basic Docker Compose file would look like this:
```yaml
version: "3"
services:
  db:
    image: postgres:12
    volumes:
      - mattermost-db:/var/lib/postgresql/data
  app:
    image: mattermost/mattermost-team-edition:latest
    environment:
      - MM_USERNAME=youradmin
      - MM_PASSWORD=yourpassword
      - MM_DB_HOST=db:5432
    ports:
      - "8065:8065"
    depends_on:
      - db
volumes:
  mattermost-db:
```
You would need to configure your Django services to send alerts via REST API or WebSocket to Mattermost channels.

### 2. **Rocket.Chat**
**Rocket.Chat** is another powerful open-source self-hosted messaging platform, widely used as an alternative to Slack and Discord. It has rich features, supports REST APIs, WebSockets, and can be containerized with Docker.

- **Docker Support**: Rocket.Chat offers official Docker images.
- **Real-time Messaging**: It uses WebSockets for real-time messaging, which will allow you to display live alerts on your monitor.
- **Extensibility**: It comes with a REST API and can be integrated easily with your Django backend to send alerts.
- **Features**:
  - Channels, private messages, and direct messages.
  - Integration with many other services (can be useful for integrations in your case).
  - Web and mobile apps that could be customized.
  - Real-time updates via WebSocket.

**Setup with Docker:**
Here’s a simple `docker-compose.yml` to run Rocket.Chat with MongoDB:
```yaml
version: '3'
services:
  rocketchat:
    image: rocketchat/rocket.chat:latest
    environment:
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - ROOT_URL=http://localhost:3000
      - PORT=3000
    ports:
      - "3000:3000"
    depends_on:
      - mongo
  mongo:
    image: mongo:4.0
    volumes:
      - ./data/db:/data/db
volumes:
  data:
    driver: local
```
To integrate with your Django services, you can use Rocket.Chat's REST API to post messages.

### 3. **Zulip**
**Zulip** is an open-source chat platform that's designed for team collaboration. It has a unique "stream" and "topic" model, which could be useful for grouping alerts and messages into different categories or topics. Zulip supports real-time messaging, WebSockets, and has a well-documented API for integration with other services.

- **Docker Support**: Zulip provides official Docker images and even a full Docker Compose setup for easy deployment.
- **Real-time Messaging**: It supports WebSockets for real-time communication, making it suitable for live notifications on your wall-mounted screen.
- **Extensibility**: Zulip offers both REST APIs and WebSockets for sending messages from your Django backend to Zulip.
- **Features**:
  - Streams and topics to organize messages.
  - Direct messages and group messages.
  - Integration with external tools (e.g., webhooks).
  - Mobile and desktop clients.

**Setup with Docker:**
You can deploy Zulip with Docker using their provided instructions for quick setup:
```bash
curl -sSL https://raw.githubusercontent.com/zulip/zulip/master/tools/provision.py | python3
```
This will set up everything needed to get Zulip running in Docker.

### 4. **Matrix (with Synapse and Riot/Element)**
**Matrix** is an open-source protocol for decentralized communication, and Synapse is its reference server implementation. **Element (formerly Riot)** is the client for Matrix. This setup would provide you with a decentralized, federated messaging platform.

- **Docker Support**: Matrix (Synapse) and Element have Docker images available for quick deployment.
- **Real-time Messaging**: Matrix natively supports real-time communication, and it works well with WebSockets.
- **Extensibility**: Matrix offers a rich REST API and webhook support, which allows you to send messages from Django services.

**Setup with Docker:**
Here’s a basic `docker-compose.yml` setup for Matrix Synapse:
```yaml
version: '3'
services:
  synapse:
    image: matrixdotorg/synapse:latest
    ports:
      - "8008:8008"
    environment:
      - SYNAPSE_SERVER_NAME=localhost
      - SYNAPSE_REPORT_STATS=no
    volumes:
      - synapse_data:/data
  nginx:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
volumes:
  synapse_data:
    driver: local
```
You can use Element as the client to display messages.

### Integration with Django
Once you set up the server, you'll need to integrate your Django Rest Framework services to send alerts to these platforms. Here’s a general approach:

1. **Sending Alerts via WebSocket or REST API**:
   - For **Mattermost**, **Rocket.Chat**, and **Zulip**, you'll primarily use their REST APIs to send messages.
   - For **Matrix**, you can use their REST API or directly send events through WebSockets.

2. **Django Integration Example** (for a REST API):
   ```python
   import requests
   
   def send_alert_to_chat(message, room_id):
       url = "https://your-chat-server.com/api/v1/messages"
       headers = {
           "Authorization": "Bearer your_api_token",
           "Content-Type": "application/json"
       }
       payload = {
           "room_id": room_id,
           "text": message
       }
       response = requests.post(url, json=payload, headers=headers)
       return response.status_code
   ```
   You can call this function whenever your Django microservices need to send an alert.

### Conclusion
- **Mattermost** and **Rocket.Chat** are the best open-source alternatives for your use case, with easy Docker setups and good API support.
- **Zulip** is also a strong option if you're interested in a more organized message system.
- **Matrix** offers a decentralized solution, but it might be more complex to set up.

All of these systems can be integrated with your Django-based services for real-time alerts, and each has a Docker-friendly environment. Based on your needs for scalability, extensibility, and ease of use, I recommend starting with **Mattermost** or **Rocket.Chat**.
=======================================
Yes, it’s absolutely possible to use Rocket.Chat's WebSocket API to implement a "sign of life" (heartbeat) system for your microservices. You can create a separate user (or bot account) for each of your microservices and monitor their connection status in Rocket.Chat using different indicators (e.g., green for active, yellow for error, and red for down). Below is a detailed approach to how you can achieve this.

### Key Steps for Implementing the "Sign of Life" System:

1. **Create User Accounts for Each Microservice**:
   - For each microservice, create a user or bot account in Rocket.Chat. This user will represent the microservice and will send WebSocket pings (heartbeat messages) to indicate its status.
   - You can do this manually through the Rocket.Chat admin interface or automate the creation of users via the Rocket.Chat REST API.

2. **Use WebSocket to Maintain Connection**:
   - Your microservices will open a WebSocket connection to the Rocket.Chat server. WebSockets are perfect for maintaining persistent real-time communication, and they can be used to send heartbeat signals (sign of life) periodically.

3. **Sending Heartbeats via WebSocket**:
   - You can configure the microservices to send regular messages (heartbeats) over WebSockets. The microservice will send a "heartbeat" message every few seconds or minutes (depending on your desired frequency).
   - For example, each microservice would send a ping message through the WebSocket connection to notify that it's still alive.

4. **Monitoring the WebSocket Connection**:
   - On the Rocket.Chat side, you can monitor the WebSocket connection to check whether a microservice is still sending heartbeats.
   - If a microservice stops sending heartbeats, you can update the status of the corresponding user in Rocket.Chat (e.g., change the color of the user's indicator).

5. **Managing Status Updates (Green, Yellow, Red)**:
   - **Green**: If the microservice is sending regular heartbeats and the WebSocket connection is healthy, show a green indicator.
   - **Yellow**: If the microservice fails to send heartbeats within a certain threshold (e.g., 5 minutes) or there is an error in the WebSocket connection, show a yellow warning indicator.
   - **Red**: If the WebSocket connection is completely down or the microservice hasn’t sent a heartbeat for an extended period, show a red indicator.

### Implementation Outline

#### 1. **Create Users for Microservices**
You can create a user for each microservice in Rocket.Chat manually or using the API. For example, to create a user via the REST API:

```bash
curl -X POST \
  http://your_rocket_chat_url/api/v1/users.create \
  -H "X-Auth-Token: your_auth_token" \
  -H "X-User-Id: your_user_id" \
  -d '{"email": "microservice1@yourdomain.com", "username": "microservice1", "password": "securepassword", "name": "Microservice 1"}'
```

You can do the same for each microservice, generating unique usernames and emails for each.

#### 2. **WebSocket Connection and Heartbeat Logic**

To set up the WebSocket connection and heartbeat logic, your microservices need to connect to Rocket.Chat using the WebSocket API. You’ll send a heartbeat signal at regular intervals (e.g., every 30 seconds).

Here’s an example in Python to establish the WebSocket connection and send heartbeats:

```python
import websocket
import json
import time

def on_message(ws, message):
    print(f"Received message: {message}")

def on_error(ws, error):
    print(f"Error: {error}")

def on_close(ws, close_status_code, close_msg):
    print("Connection closed")

def on_open(ws):
    print("Connection established")
    
    # Send heartbeat every 30 seconds
    while True:
        heartbeat_msg = {
            "msg": "ping",
            "id": "heartbeat_id"
        }
        ws.send(json.dumps(heartbeat_msg))  # Send a ping message
        time.sleep(30)  # Send ping every 30 seconds

def connect_to_rocket_chat():
    # WebSocket URL for Rocket.Chat server (adjust if using SSL)
    ws_url = "ws://your_rocket_chat_url/websocket"
    
    # Establish WebSocket connection
    ws = websocket.WebSocketApp(ws_url,
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever()

if __name__ == "__main__":
    connect_to_rocket_chat()
```

This script opens a WebSocket connection to Rocket.Chat and sends a heartbeat (`ping`) every 30 seconds.

#### 3. **Handling the "Sign of Life" with Status Updates**
In Rocket.Chat, you can modify the user’s status based on the received heartbeat messages. There’s no native feature to automatically update the presence status of users based on external WebSocket activity, but you can implement it by interacting with the API. For example, when the microservice stops sending heartbeats, you can change the status using the Rocket.Chat REST API.

For example, after a certain timeout threshold, you can change the user’s status to indicate "disconnected":

```bash
curl -X POST \
  http://your_rocket_chat_url/api/v1/users.setStatus \
  -H "X-Auth-Token: your_auth_token" \
  -H "X-User-Id: your_user_id" \
  -d '{"userId": "microservice1_id", "status": "away"}'
```

You could also change the user's status color by using Rocket.Chat's presence or status settings.

#### 4. **Visualizing in the Client Application**
In the Rocket.Chat client (web or mobile), the status of each user (or bot representing a microservice) can be visually represented by their presence indicator:

- **Green**: If the microservice is connected and sending heartbeats, the presence indicator will be green.
- **Yellow/Orange**: If the microservice is not sending heartbeats within the allowed time, the indicator will change to yellow.
- **Red**: If the WebSocket connection is completely down, the indicator can be set to red.

You can also create a custom dashboard in Rocket.Chat that shows a list of all the microservices and their respective statuses. This could be achieved using Rocket.Chat’s `rooms`, `channels`, or custom integration within Rocket.Chat to display the health of the system.

#### 5. **Error Handling and Monitoring**
You can create a simple monitoring script to detect when the WebSocket connection drops or fails to send heartbeats and take appropriate action to change the status in Rocket.Chat. For example, if a microservice fails to send a heartbeat within a certain timeout period, you can mark it as "offline."

---

### Final Thoughts
- **WebSocket Heartbeats**: Microservices can send WebSocket pings at regular intervals (e.g., every 30 seconds) to indicate they are alive.
- **User Status in Rocket.Chat**: Use the Rocket.Chat API to change the user’s presence or status based on whether the heartbeat is received or the WebSocket connection is alive.
- **Visual Indicators**: Rocket.Chat clients will show a presence indicator for each user, which can be colored based on the service’s state (green, yellow, red).

By following this approach, you can effectively monitor the health of each of your microservices in real-time using Rocket.Chat's WebSocket API and user status indicators.

===============================================
To create a monitoring microservice using Django REST Framework (DRF) that detects when a WebSocket connection drops or fails to send heartbeats, and subsequently updates the status of the microservice in Rocket.Chat, we can break the task into the following steps:

### Steps:
1. **Create a Django application for monitoring microservices**.
2. **Implement a WebSocket client within the microservice** that monitors heartbeat responses.
3. **Track the heartbeat status** of each microservice.
4. **Implement error handling** and automatically update the microservice status to "offline" if no heartbeat is received in time.
5. **Use the Rocket.Chat API** to update the status of the microservice in Rocket.Chat (for example, changing it to "offline" when the WebSocket connection drops or when no heartbeats are received).

### Solution Breakdown

#### 1. **Create a Django App for Monitoring Microservices**
First, you need to create a Django app that will monitor the microservices. This app will handle the communication with Rocket.Chat's API to update user status based on the heartbeat status of each microservice.

```bash
python manage.py startapp monitoring
```

In the `monitoring` app, you’ll create models (optional) to track the heartbeat status, views to trigger updates, and possibly a Celery task to monitor the heartbeat status periodically.

#### 2. **Install Dependencies**
You will need the following Python packages:
- `websocket-client` for the WebSocket connection to the Rocket.Chat server.
- `requests` for interacting with Rocket.Chat's REST API to update the status.

```bash
pip install websocket-client requests
```

#### 3. **Create a WebSocket Client for Each Microservice**

To monitor the heartbeat status, you’ll need to establish a WebSocket connection in the microservice. Here's how to implement a simple WebSocket client in Python.

##### Example: WebSocket Client to Send Heartbeats

In your monitoring service, you’ll need to create a WebSocket client that connects to Rocket.Chat and sends periodic heartbeats. The client should check for incoming messages, track heartbeats, and reset the heartbeat timer each time a message is received.

```python
import websocket
import json
import time
import threading

# Global dictionary to track the last received heartbeat for each microservice
microservice_heartbeat = {}

# Function to update the status of the microservice in Rocket.Chat
def update_status_to_rocket_chat(microservice_id, status):
    # Rocket.Chat API credentials (replace with your actual API token and user ID)
    rocket_chat_url = 'http://your_rocket_chat_server/api/v1/users.setStatus'
    auth_token = 'your_auth_token'
    user_id = 'your_user_id'
    
    # The status can be 'online', 'away', or 'offline' based on the heartbeat status
    status_data = {
        'userId': microservice_id,
        'status': status
    }
    response = requests.post(
        rocket_chat_url,
        headers={
            'X-Auth-Token': auth_token,
            'X-User-Id': user_id
        },
        json=status_data
    )
    
    if response.status_code == 200:
        print(f"Updated {microservice_id} status to {status}")
    else:
        print(f"Failed to update {microservice_id} status: {response.text}")

# Function to handle WebSocket messages and detect heartbeats
def on_message(ws, message):
    global microservice_heartbeat
    print(f"Received message: {message}")
    
    # Assume that message contains a 'heartbeat' signal with a timestamp or ping identifier
    heartbeat_msg = json.loads(message)
    if heartbeat_msg.get('msg') == 'ping':
        microservice_heartbeat[heartbeat_msg['id']] = time.time()  # Update the last heartbeat timestamp

# Function to handle WebSocket errors
def on_error(ws, error):
    print(f"Error: {error}")

# Function to handle WebSocket close events
def on_close(ws, close_status_code, close_msg):
    print("WebSocket connection closed")

# Function to open WebSocket connection and start heartbeat monitoring
def on_open(ws):
    print("WebSocket connection established")
    
    # Send a heartbeat (ping) every 30 seconds
    while True:
        heartbeat_msg = {
            "msg": "ping",
            "id": "microservice1"  # Unique ID for the microservice
        }
        ws.send(json.dumps(heartbeat_msg))
        time.sleep(30)  # Send ping every 30 seconds

# Function to start the WebSocket client and monitor heartbeat status
def start_websocket_monitoring(microservice_id):
    ws_url = "ws://your_rocket_chat_server/websocket"
    
    ws = websocket.WebSocketApp(
        ws_url,
        on_message=on_message,
        on_error=on_error,
        on_close=on_close
    )
    
    ws.on_open = on_open
    ws.run_forever()

# Start the WebSocket client in a separate thread for monitoring
def start_monitoring_thread(microservice_id):
    thread = threading.Thread(target=start_websocket_monitoring, args=(microservice_id,))
    thread.start()

# Function to check heartbeat timeout and update status if necessary
def check_heartbeat_status():
    while True:
        for microservice_id, last_heartbeat_time in list(microservice_heartbeat.items()):
            current_time = time.time()
            timeout_threshold = 60  # 60 seconds timeout threshold for heartbeat

            if current_time - last_heartbeat_time > timeout_threshold:
                print(f"Microservice {microservice_id} has timed out")
                # Mark the microservice as offline in Rocket.Chat
                update_status_to_rocket_chat(microservice_id, 'offline')
            else:
                print(f"Microservice {microservice_id} is online")
                # Mark the microservice as online in Rocket.Chat
                update_status_to_rocket_chat(microservice_id, 'online')
        
        time.sleep(30)  # Check every 30 seconds for heartbeat timeout

if __name__ == "__main__":
    # Start monitoring the microservices
    start_monitoring_thread("microservice1")

    # Periodically check heartbeat status and update Rocket.Chat status
    check_heartbeat_status()
```

### Explanation:
1. **WebSocket Client**: 
   - The WebSocket client connects to Rocket.Chat and sends a "ping" message (heartbeat) every 30 seconds.
   - It listens for incoming messages (from Rocket.Chat or other services) and updates the heartbeat timestamp.
   
2. **Heartbeat Monitoring**: 
   - The `microservice_heartbeat` dictionary stores the last received heartbeat time for each microservice.
   - The `check_heartbeat_status` function runs in the background, checking if the microservices have timed out (i.e., they haven’t sent a heartbeat within the timeout threshold).
   - If a microservice fails to send a heartbeat in time, it updates the status to "offline" in Rocket.Chat via the `update_status_to_rocket_chat` function.
   
3. **Threading**:
   - A separate thread is created to run the WebSocket client to monitor each microservice. You can start monitoring multiple microservices by calling `start_monitoring_thread()` for each microservice ID.

4. **Error Handling**:
   - If the WebSocket connection is dropped or if an error occurs, it will be caught by the `on_error` and `on_close` functions.
   
#### 4. **Integrating with Rocket.Chat's API**
- The function `update_status_to_rocket_chat` uses Rocket.Chat's REST API to change the user status to "online" or "offline" based on the heartbeat response.
- You can customize the status (e.g., "away", "offline", or "online") as needed.

### 5. **Running the Monitoring Service**
To run this monitoring service, include it in your Django views or use a periodic task scheduler like **Celery** to periodically check the status of the microservices. Here's an example using Django views:

#### Example Django View:
```python
# monitoring/views.py

from django.http import JsonResponse
from django.views import View
from .monitoring_service import start_monitoring_thread

class StartMonitoringView(View):
    def get(self, request, microservice_id):
        # Start the WebSocket monitoring thread for the given microservice
        start_monitoring_thread(microservice_id)
        return JsonResponse({"status": "started", "microservice_id": microservice_id})
```

#### URL Routing:
```python
# monitoring/urls.py

from django.urls import path
from .views import StartMonitoringView

urlpatterns = [
    path('monitor/microservice/<str:microservice_id>/', StartMonitoringView.as_view(), name='start-monitoring'),
]
```

This view will start the WebSocket monitoring when you access it through the endpoint `/monitor/microservice/<microservice_id>/`.

### Conclusion
- **WebSocket Client**: Each microservice has a WebSocket client that sends regular heartbeats.
- **Django Monitoring Service**: The Django service monitors the heartbeat and updates Rocket.Chat status accordingly.
- **Timeout Handling**: If a heartbeat is not received within the threshold, the status is updated to "offline".
  
This monitoring service should help you track the health of each microservice and reflect the status in Rocket.Chat in real-time.

======================================================
That sounds like a great approach! By creating a dedicated Django REST Framework (DRF) microservice to handle the communication between your existing microservices and Rocket.Chat, you can centralize the processing and persistence of messages in MongoDB, while still ensuring real-time communication with Rocket.Chat via WebSockets.

### The System Architecture

1. **Incoming Messages from Microservices**: 
   - Your existing microservices will send messages via WebSockets to the new DRF monitoring microservice.
   - The DRF service will store these messages in MongoDB (or any other database of your choice).
   
2. **Persistence in MongoDB**: 
   - The DRF microservice will persist the incoming messages in MongoDB for future reference, auditing, and possible replay of messages.

3. **Forwarding to Rocket.Chat**: 
   - After receiving and persisting messages, the DRF service will forward the messages to Rocket.Chat via WebSocket (using Rocket.Chat's WebSocket API) for real-time communication.

4. **Status Management**: 
   - Based on certain conditions (e.g., missing heartbeats, service downtime), the DRF microservice can update the status of the microservices in Rocket.Chat.

### Steps to Implement

1. **Create a New Django Microservice**: This service will act as the gateway between the microservices and Rocket.Chat.
2. **Set Up MongoDB Database**: This database will store all messages for persistence and audit purposes.
3. **Implement WebSocket Clients**: The microservice will handle incoming WebSocket messages from microservices and forward them to Rocket.Chat.
4. **Handle Message Persistence**: All incoming messages will be saved in MongoDB.
5. **Send Messages to Rocket.Chat**: Messages will be sent to Rocket.Chat through the WebSocket API.
6. **Error Handling and Status Updates**: Monitor the health of connections and update microservice statuses in Rocket.Chat.

### Detailed Steps

#### 1. **Set Up Your Django Microservice**

First, you'll need to create a Django project and configure MongoDB. You can use `djongo` or `mongoengine` to integrate MongoDB with Django.

1. **Install dependencies**:
   ```bash
   pip install django djangorestframework djongo websocket-client requests
   ```

2. **Create a Django Project**:
   ```bash
   django-admin startproject rocket_chat_gateway
   cd rocket_chat_gateway
   python manage.py startapp message_gateway
   ```

3. **Configure MongoDB**:
   In your Django project’s `settings.py`, configure MongoDB:

   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'djongo',
           'NAME': 'rocket_chat_messages',  # MongoDB database name
       }
   }
   ```

   If you prefer `mongoengine`, the configuration is as follows:
   ```python
   DATABASES = {
       'default': {
           'ENGINE': 'django.db.backends.dummy',  # Dummy backend for MongoDB
       }
   }

   # MongoDB Connection Configuration
   MONGO_DB = {
       'db': 'rocket_chat_messages',
       'host': 'mongodb://localhost:27017',
   }
   ```

#### 2. **Create MongoDB Models for Message Persistence**

Create a model to store the incoming messages from the microservices.

```python
# message_gateway/models.py

from djongo import models

class Message(models.Model):
    sender = models.CharField(max_length=100)  # Sender's identifier (e.g., microservice name)
    message = models.TextField()  # The content of the message
    timestamp = models.DateTimeField(auto_now_add=True)  # Timestamp for when the message was received

    class Meta:
        verbose_name = 'Message'
        verbose_name_plural = 'Messages'

    def __str__(self):
        return f"Message from {self.sender} at {self.timestamp}"
```

#### 3. **WebSocket Client for Receiving Messages**

Create a WebSocket client within your DRF service that will listen for messages from other microservices.

```python
# message_gateway/websocket_client.py

import websocket
import json
import threading
from .models import Message
from datetime import datetime

# The WebSocket server URL for your microservices
WEB_SOCKET_SERVER_URL = 'ws://your_microservice_url'

def on_message(ws, message):
    """
    Handle incoming messages from microservices.
    Save the messages in the MongoDB database and forward them to Rocket.Chat.
    """
    print(f"Received message: {message}")

    # Persist the message to MongoDB
    try:
        message_data = json.loads(message)
        Message.objects.create(
            sender=message_data.get('sender'),
            message=message_data.get('message')
        )
        print(f"Message saved from {message_data['sender']}")
    except Exception as e:
        print(f"Error saving message: {e}")
    
    # Forward the message to Rocket.Chat
    forward_to_rocket_chat(message_data)

def on_error(ws, error):
    print(f"Error: {error}")

def on_close(ws, close_status_code, close_msg):
    print("WebSocket connection closed")

def on_open(ws):
    print("WebSocket connection established")

def start_websocket_client():
    """
    Connect to the WebSocket server to receive messages from microservices.
    """
    ws = websocket.WebSocketApp(
        WEB_SOCKET_SERVER_URL,
        on_message=on_message,
        on_error=on_error,
        on_close=on_close
    )
    ws.on_open = on_open
    ws.run_forever()

# Run the WebSocket client in a separate thread
def start_monitoring():
    thread = threading.Thread(target=start_websocket_client)
    thread.start()
```

#### 4. **Forwarding Messages to Rocket.Chat**

Now, implement the function to forward the message from the microservice to Rocket.Chat.

```python
# message_gateway/rocket_chat_api.py

import requests
import json

# Rocket.Chat API URL and credentials
ROCKET_CHAT_URL = 'http://your_rocket_chat_server/api/v1/chat.postMessage'
ROCKET_CHAT_AUTH_TOKEN = 'your_rocket_chat_auth_token'
ROCKET_CHAT_USER_ID = 'your_rocket_chat_user_id'

def forward_to_rocket_chat(message_data):
    """
    Forward the received message to Rocket.Chat using its WebSocket API.
    """
    message = message_data.get('message')
    sender = message_data.get('sender')

    data = {
        "channel": "#general",  # You can change this to any Rocket.Chat channel
        "text": f"Message from {sender}: {message}",
        "alias": sender
    }

    headers = {
        'X-Auth-Token': ROCKET_CHAT_AUTH_TOKEN,
        'X-User-Id': ROCKET_CHAT_USER_ID
    }

    response = requests.post(ROCKET_CHAT_URL, headers=headers, data=data)

    if response.status_code == 200:
        print("Message successfully forwarded to Rocket.Chat.")
    else:
        print(f"Error forwarding message to Rocket.Chat: {response.text}")
```

#### 5. **Set Up Periodic Message Monitoring**

Use Django's **Celery** or **Django management commands** to periodically check for heartbeats or service failures. If a microservice fails to send a message within a certain timeframe, you can update its status.

For example, with **Celery**, you could define a periodic task to check message timestamps and mark the microservice as offline after a certain timeout.

#### 6. **Start WebSocket Client in Django** 

To start the WebSocket client when the server starts, you can either use **Django management commands** or start the client in a background thread. Here’s an example of running the WebSocket client automatically when the Django app starts:

```python
# message_gateway/apps.py

from django.apps import AppConfig

class MessageGatewayConfig(AppConfig):
    name = 'message_gateway'

    def ready(self):
        from .websocket_client import start_monitoring
        start_monitoring()  # Start WebSocket client when the app is ready
```

In the `INSTALLED_APPS` section of your `settings.py`, make sure `message_gateway` is listed.

#### 7. **Run the Service**

Once everything is set up, you can run the Django development server or deploy it using a production-ready server like Gunicorn or Daphne, depending on your environment.

```bash
python manage.py runserver
```

---

### Final Thoughts

1. **Centralized Message Gateway**: The new DRF microservice serves as a centralized gateway for handling WebSocket communication, message persistence in MongoDB, and forwarding messages to Rocket.Chat.
2. **Message Persistence**: All incoming messages from your microservices are stored in MongoDB for auditing or replay purposes.
3. **Real-Time Communication**: The WebSocket client listens for incoming messages and immediately forwards them to Rocket.Chat for real-time communication.
4. **Status Updates**: If a microservice fails to send messages in a timely manner, you can handle status updates to reflect this in Rocket.Chat.

By building this system, you create a robust solution that decouples your microservices from direct interaction with Rocket.Chat, adds message persistence, and centralizes error handling and monitoring.