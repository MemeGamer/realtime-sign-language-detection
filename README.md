<p align="center"><img src="docs/cover.jpg" alt="Illustration of a monitor showing a hand gesture with landmark dots and distance lines between fingertips, a face placeholder box, and a speaker beside it" width="100%"></p>

# Realtime Sign Language Detection

Rule-based recognition of a small set of hand signs from a webcam, using MediaPipe hand landmarks and distance thresholds.

## What it does

- Tracks one hand on the webcam and labels the current gesture on screen.
- **Words and phrases** (`SignLanguageMaster.py`): ILY, YOO, GOOD JOB, I, COME HERE, HI, Doing, You, Are, What, How are you, Want, Food.
- Builds two simple sentences when signs are made in order: "What Are You Doing" and "HI, COME HERE".
- Detects the face with a Haar cascade so that signs made near the mouth ("Food") can be told apart from the same hand shape elsewhere ("Want").
- **Letters** (`Sign_language_Letters.py`): L, E, H, A, O, I, V.
- Shows FPS and a rough "accuracy" hint based on how close the wrist is to the centre of the frame.
- A Tkinter launcher (`GUI.py`) offers buttons for the words mode, the letters mode and exit.

## How it works

`HandTrackingModule.py` wraps MediaPipe Hands and returns the 21 hand landmarks as pixel coordinates. The detectors then compute Euclidean distances (`math.hypot`) between chosen landmarks, for example fingertip to knuckle (P8 to P5, P12 to P9) or fingertip to wrist (P8 to P0). A finger counts as extended or folded when its distance passes a fixed pixel threshold, and each sign is an `if` rule over those distances. See `hand_landmarks.png` for the landmark numbering.

In `SignLanguageMaster.py`, the distance from the middle fingertip (P12) to a point inside the detected face box is used to decide whether the hand is at the mouth.

## Tech

- Python 3
- OpenCV (`cv2`, including a Haar cascade face detector)
- MediaPipe (Hands; Pose and Face Mesh in experiment scripts)
- NumPy
- Tkinter (launcher)
- gTTS and playsound (text to speech)
- Flask (experimental `server.py`)

## Run it

Needs a webcam. Run from inside `HandTracking/`, since images and the cascade file are loaded by relative path.

```bash
pip install opencv-python mediapipe numpy gTTS playsound
cd HandTracking
python SignLanguageMaster.py      # words, phrases and sentences
python GUI.py                     # Tkinter launcher for words or letters
```

## Project structure

- `HandTracking/SignLanguageMaster.py`: main script with word, phrase, face-aware and sentence logic.
- `HandTracking/SignLaguage.py`: earlier words detector as a `signlanguage()` function, used by the GUI.
- `HandTracking/Sign_language_Letters.py`: letters detector as a `letters()` function, used by the GUI.
- `HandTracking/GUI.py`: Tkinter launcher with image buttons.
- `HandTracking/HandTrackingModule.py`: MediaPipe hand detector class.
- `HandTracking/TextToSpeech.py`: speaks a detected label with gTTS and playsound.
- `HandTracking/BodyDetection.py`: MediaPipe Pose webcam demo.
- `HandTracking/HandTrackingMin.py`, `test.py`, `test2.py`: experiments (minimal hand tracking, an alternative detector, face mesh plus hands).
- `HandTracking/server.py`: unfinished Flask endpoint.
- `HandTracking/main.py`: default PyCharm template, unused.
- `HandTracking/haarcascade_frontalface_default.xml`: OpenCV face cascade.
- `HandTracking/*.png`, `fingerhand.ico`: GUI images and the landmark reference chart.
- `HandTracking/output.mp3`: leftover text-to-speech output.

## Notes

- Recognition is hand-written thresholds in pixels, not a trained model, so it depends on the distance to the camera and the 640x480 resolution. The on-screen "Accuracy" text is only a wrist-position hint, not a measured accuracy.
- Text to speech is wired up but mostly commented out; where it runs, it blocks the video loop while audio plays and needs an internet connection.
- `HandTrackingMin.py` does not run as written (`mp.zeros` and `cv2.COLOR_BGR2` are not valid).
- `server.py` calls `exec('SignLanguage.py')` on a misspelled file name and does not serve the detector.
- None of the loops have a quit key; stop them with Ctrl+C.
- `__pycache__/` is committed.
- MediaPipe settings are passed positionally to `Hands()`; newer MediaPipe versions may need keyword arguments or a 0.8.x install.
