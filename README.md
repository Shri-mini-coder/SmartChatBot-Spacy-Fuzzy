# SmartChatBot-Spacy-Fuzzy
Training for project on Implementation of chatbot using NLP
import random
import spacy
import re
from textblob import TextBlob
from fuzzywuzzy import fuzz

# Load spaCy NLP model
nlp = spacy.load("en_core_web_sm")

# Dictionary for chatbot responses
responses = {
    "greeting": ["Hello! How can I help?", "Hey there! 😊", "Hi! What’s on your mind?"],
    "goodbye": ["Goodbye! Have a great day!", "See you later! Take care!", "Bye! Looking forward to our next chat!"],
    "thanks": ["You're welcome!", "Anytime! 😊", "Glad I could help!"],
    "about": ["I'm an AI chatbot designed to help you. Ask me anything!", "I’m here to assist you with any questions!"],
    "help": ["Sure! Let me know how I can help.", "I’m here to assist. What do you need help with?"],
    "weather": ["I don’t have real-time weather data. Try checking a weather app!", "You can check the weather online!"],
    "jokes": ["Why don’t programmers like nature? Too many bugs! 😂", "Why did the chatbot break up? It needed more space! 🚀"],
    "motivation": ["Believe in yourself! 🚀 You got this!", "Keep pushing forward! 💪"],
    "unknown": ["I didn’t quite get that. Can you rephrase?", "I'm still learning! Could you clarify?"],
}

# Memory to store user details
user_memory = {
    "name": None,
    "interests": [],
    "last_topic": None,
}

# Function to extract user's name
def extract_name(user_input):
    doc = nlp(user_input)
    for ent in doc.ents:
        if ent.label_ == "PERSON":  # Detect names
            user_memory["name"] = ent.text
            return f"Nice to meet you, {ent.text}!"
    return None

# Function to analyze sentiment
def analyze_sentiment(text):
    blob = TextBlob(text)
    sentiment = blob.sentiment.polarity  # Get sentiment polarity
    if sentiment > 0.3:
        return "You seem happy! 😊"
    elif sentiment < -0.3:
        return "I sense you're feeling a bit down. I’m here for you. 🤗"
    return None  # Return None instead of default response

# Function to find the best matching response
def get_best_match(user_input):
    highest_score = 0
    best_match = "unknown"

    for key in responses.keys():
        score = fuzz.partial_ratio(user_input, key)
        if score > highest_score:
            highest_score = score
            best_match = key

    return best_match

# Function to handle chatbot response
def chatbot_response(user_input):
    user_input = user_input.lower()

    # Check if user is providing their name
    name_response = extract_name(user_input)
    if name_response:
        return name_response

    # Fuzzy match best response
    best_match = get_best_match(user_input)

    # Store last topic for context-aware conversations
    user_memory["last_topic"] = best_match

    # Check for sentiment only if no predefined response is found
    if best_match == "unknown":
        sentiment_response = analyze_sentiment(user_input)
        if sentiment_response:
            return sentiment_response

    # Generate response
    return random.choice(responses.get(best_match, responses["unknown"]))

# Main chatbot loop
def chatbot():
    print("Chatbot: Hello! What's your name? (Type 'bye' to exit)")

    while True:
        user_input = input("You: ")

        if user_input.lower() == 'bye':
            print(f"Chatbot: Goodbye {user_memory['name']}! Have a great day!" if user_memory["name"] else "Chatbot: Goodbye! Have a great day!")
            break

        response = chatbot_response(user_input)
        print(f"Chatbot: {response}")

# Run chatbot
chatbot()
