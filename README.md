# AI-chatbot-for-budget-Hotel
 have budget hotel with 300 rooms. We are using Respond.io for answering the guest's inquiries through WhatsApp. Nowadays, the guest number is increasing and it is required to hire more customers services team and really hard to maintain... Therefore, I need to apply AI chat to answer to the guests more faster using my knowledge based and also using my database to retrieve the reservation details. 
----------------
To automate the customer service for your hotel using AI, leveraging Respond.io for WhatsApp messaging and integrating a knowledge base along with reservation data retrieval, you can create a solution using OpenAI's GPT model (or similar), along with API integration for your hotel's reservation system.

Here’s a high-level approach to solve the problem:

    Connect to Respond.io: Use Respond.io API to receive messages from guests and respond to them in real-time through WhatsApp.
    Integrate AI (GPT-3/4): Use OpenAI's GPT models to process guest inquiries and generate responses based on the knowledge base.
    Database Querying for Reservation Details: Query your reservation database to retrieve booking information when requested by guests (e.g., booking status, room type, check-in/check-out dates).
    Conversation Flow: Build a conversation flow that can intelligently handle FAQs, check reservation status, offer hotel amenities, and assist with booking changes.

Components of the Solution:

    Respond.io API Integration: To send/receive WhatsApp messages.
    AI Chatbot (GPT): To generate intelligent responses.
    Hotel Database: To retrieve reservation details.
    Flask Web Server: For handling API requests and processing logic.

Key Requirements:

    Python for backend.
    Flask for handling the API.
    OpenAI API for the AI chatbot.
    Database (MySQL/PostgreSQL) for reservation data storage and retrieval.
    Respond.io API for WhatsApp integration.

Solution Architecture:

    WhatsApp Message Reception: Respond.io API receives the messages from the guests.
    Message Parsing and Intent Recognition: The message will be passed to the AI for analysis (via GPT).
    Fetch Reservation Details: If the message contains a request for reservation details (e.g., booking status, dates, etc.), query the hotel’s database for the relevant information.
    Responding to the User: Generate the appropriate response using GPT or pull data directly from the database (if required).

Python Code Implementation
Step 1: Install Required Libraries

First, make sure to install the required Python libraries:

pip install openai flask mysql-connector respondio

Step 2: Set Up the Backend (Flask Server)

Here’s an implementation using Flask to handle WhatsApp messages and connect to OpenAI for AI responses, while also querying a database for reservation information.

import openai
import mysql.connector
from flask import Flask, request, jsonify
import requests

# Initialize Flask app
app = Flask(__name__)

# OpenAI API key for AI-based responses
openai.api_key = 'your_openai_api_key_here'

# MySQL Database connection settings (make sure to configure this based on your DB)
db_config = {
    'host': 'your_db_host',
    'user': 'your_db_user',
    'password': 'your_db_password',
    'database': 'your_database_name'
}

# Respond.io API settings (use your own access token and endpoint)
RESPOND_IO_API_URL = "https://api.respond.io/v1/messages"
RESPOND_IO_ACCESS_TOKEN = "your_respond_io_access_token"

# Function to send message to Respond.io API (WhatsApp)
def send_whatsapp_message(to, message):
    payload = {
        "to": to,
        "text": message
    }
    headers = {
        "Authorization": f"Bearer {RESPOND_IO_ACCESS_TOKEN}",
        "Content-Type": "application/json"
    }
    response = requests.post(RESPOND_IO_API_URL, json=payload, headers=headers)
    return response.json()

# Function to query the reservation details from the database
def get_reservation_details(guest_id):
    try:
        # Connect to the database
        connection = mysql.connector.connect(**db_config)
        cursor = connection.cursor(dictionary=True)
        
        # Query the reservation details
        query = "SELECT * FROM reservations WHERE guest_id = %s"
        cursor.execute(query, (guest_id,))
        reservation = cursor.fetchone()
        
        # Close the database connection
        cursor.close()
        connection.close()
        
        if reservation:
            return reservation
        else:
            return "No reservation found with that guest ID."
    
    except mysql.connector.Error as err:
        return f"Database error: {err}"

# Function to handle incoming WhatsApp messages
@app.route('/webhook', methods=['POST'])
def handle_incoming_message():
    # Extract message data from the incoming request
    data = request.get_json()
    sender_number = data['sender']
    incoming_message = data['message']
    
    # Check if the message contains a guest's reservation query
    if "reservation" in incoming_message.lower():
        # Assuming the guest's ID is provided in the message (you can modify this to handle other ways of identifying guests)
        guest_id = incoming_message.split()[-1]  # Example: "reservation 12345"
        reservation_details = get_reservation_details(guest_id)
        
        response_message = f"Reservation details: {reservation_details}"
        send_whatsapp_message(sender_number, response_message)
    
    else:
        # Process the message through the GPT model to generate a response
        response_message = generate_ai_response(incoming_message)
        send_whatsapp_message(sender_number, response_message)
    
    return jsonify({"status": "success"}), 200

# Function to generate a response using OpenAI's GPT model
def generate_ai_response(user_message):
    try:
        prompt = f"The user is asking: {user_message}. Provide an appropriate and polite response for a hotel guest."
        
        response = openai.Completion.create(
            model="text-davinci-003",
            prompt=prompt,
            temperature=0.7,
            max_tokens=150
        )
        
        # Extract and return the generated response
        ai_response = response.choices[0].text.strip()
        return ai_response
    
    except Exception as e:
        return f"Sorry, there was an error processing your request: {str(e)}"

if __name__ == '__main__':
    app.run(debug=True, port=5000)

Explanation:

    Flask Web Server:
        The Flask app listens for incoming POST requests at the /webhook endpoint.
        The data from Respond.io (the guest’s WhatsApp message) is parsed to check if it contains reservation-related queries.
        If the query is related to reservation details, it pulls the guest ID from the message and queries the database for the reservation details.
        If the message is not related to a reservation, it uses OpenAI’s GPT to generate a response to the guest's inquiry.

    Respond.io API:
        send_whatsapp_message is a helper function to send messages back to the guest using the Respond.io API.
        This function posts the message payload to the Respond.io API, which then routes the message to the guest’s WhatsApp.

    Database Querying:
        The get_reservation_details function queries the reservation details from the hotel’s database (using MySQL here, but you can use any other database depending on your setup).
        The database is queried based on a guest’s unique ID or other identifying information passed through the message.

    AI Response Generation:
        The generate_ai_response function sends the user message to the OpenAI API (GPT-3/4), which generates an intelligent response for the guest.

Step 3: Running the Application

Once the Flask app is set up, you can run the server:

python app.py

Now your server should be running, and you will be able to process WhatsApp messages from Respond.io and respond with AI-generated replies or reservation details.
Final Remarks:

    This is a basic framework. You can further enhance the AI responses, improve the reservation querying process, and add more features (e.g., booking assistance, payment info, FAQs).
    You will need to implement security mechanisms to ensure that guest data is protected, especially when querying reservation details.
    Ensure that you handle error cases gracefully, especially when dealing with API limits (e.g., OpenAI API calls) and database queries.

This solution can significantly reduce the load on customer support agents by automating the most common inquiries, improving guest experience and operational efficiency.
