import openai
import speech_recognition as sr
import pyttsx3
import datetime
import pywhatkit
import wikipedia

# Initialize OpenAI API
openai.api_key = 'your-openai-api-key'

# Initialize text-to-speech engine
engine = pyttsx3.init()

def speak(text):
    engine.say(text)
    engine.runAndWait()

def listen():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        audio = r.listen(source)
    try:
        command = r.recognize_google(audio)
        print(f"User said: {command}")
        return command.lower()
    except sr.UnknownValueError:
        speak("Sorry, I didn't understand that.")
        return ""
    except sr.RequestError:
        speak("Sorry, the service is down.")
        return ""

def get_ai_response(prompt):
    try:
        response = openai.Completion.create(
            engine="text-davinci-003",
            prompt=prompt,
            max_tokens=150
        )
        return response.choices[0].text.strip()
    except Exception as e:
        return "Error getting response."

def handle_command(command):
    if "time" in command:
        time = datetime.datetime.now().strftime('%I:%M %p')
        speak(f"The time is {time}")
    elif "search" in command:
        topic = command.replace("search", "")
        info = wikipedia.summary(topic, sentences=2)
        speak(info)
    elif "play" in command:
        song = command.replace("play", "")
        speak(f"Playing {song}")
        pywhatkit.playonyt(song)
    elif "openai" in command or "question" in command:
        response = get_ai_response(command)
        speak(response)
    else:
        speak("I can’t help with that yet.")

def main():
    speak("Hello, I am your personal assistant. How can I help you today?")
    while True:
        command = listen()
        if "exit" in command or "bye" in command:
            speak("Goodbye!")
            break
        handle_command(command)

if __name__ == "__main__":
    main()
