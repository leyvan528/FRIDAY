import datetime
import platform
import subprocess
import webbrowser
import speech_recognition as sr
import pyttsx3

# Jarvis initialisieren
engine = pyttsx3.init()
engine.setProperty("rate", 180)

recognizer = sr.Recognizer()

SYSTEM = platform.system()


def sprechen(text):
    print(f"J.A.R.V.I.S.: {text}")
    engine.say(text)
    engine.runAndWait()


def zuhoeren():
    with sr.Microphone() as quelle:
        print("Ich höre zu...")

        recognizer.adjust_for_ambient_noise(
            quelle, duration=0.5
        )

        try:
            audio = recognizer.listen(
                quelle, timeout=5, phrase_time_limit=10
            )

            text = recognizer.recognize_google(
                audio, language="de-DE"
            )

            print(f"Du: {text}")
            return text.lower()

        except sr.WaitTimeoutError:
            return ""

        except sr.UnknownValueError:
            sprechen("Das habe ich leider nicht verstanden.")
            return ""

        except sr.RequestError:
            sprechen("Die Spracherkennung ist nicht erreichbar.")
            return ""


def programm_oeffnen(name):
    # Nur ausdrücklich erlaubte Programme öffnen.
    erlaubte_programme = {
        "Windows": {
            "rechner": ["calc.exe"],
            "editor": ["notepad.exe"],
        },
        "Darwin": {
            "rechner": ["open", "-a", "Calculator"],
            "editor": ["open", "-a", "TextEdit"],
        },
        "Linux": {
            "rechner": ["gnome-calculator"],
            "editor": ["gedit"],
        },
    }

    programme = erlaubte_programme.get(SYSTEM, {})
    befehl = programme.get(name)

    if not befehl:
        sprechen("Dieses Programm ist nicht freigegeben.")
        return

    try:
        subprocess.Popen(befehl)
        sprechen(f"Ich öffne den {name}.")
    except OSError:
        sprechen("Das Programm konnte nicht geöffnet werden.")


def befehl_ausfuehren(text):
    if not text:
        return True

    if "beenden" in text or "gute nacht" in text:
        sprechen("Bis später. Jarvis wird beendet.")
        return False

    elif "wie spät" in text or "uhrzeit" in text:
        jetzt = datetime.datetime.now()
        sprechen(f"Es ist {jetzt:%H Uhr %M}.")

    elif "datum" in text:
        heute = datetime.datetime.now()
        sprechen(f"Heute ist der {heute:%d.%m.%Y}.")

    elif "öffne rechner" in text or "öffne taschenrechner" in text:
        programm_oeffnen("rechner")

    elif "öffne editor" in text:
        programm_oeffnen("editor")

    elif "öffne youtube" in text:
        sprechen("Ich öffne YouTube.")
        webbrowser.open("https://www.youtube.com")

    elif "öffne google" in text:
        sprechen("Ich öffne Google.")
        webbrowser.open("https://www.google.com")

    else:
        sprechen(
            "Diesen Befehl kenne ich noch nicht. "
            "Wir können ihn später hinzufügen."
        )

    return True


def main():
    sprechen(
        "System gestartet. Guten Tag. "
        "J.A.R.V.I.S. ist einsatzbereit."
    )

    aktiv = True

    while aktiv:
        befehl = zuhoeren()
        aktiv = befehl_ausfuehren(befehl)


if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\nJarvis wurde manuell beendet.")
