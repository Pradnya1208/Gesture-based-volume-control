<div align="center">
  
[1]: https://github.com/Pradnya1208
[2]: https://www.linkedin.com/in/pradnya-patil-b049161ba/
[3]: https://public.tableau.com/app/profile/pradnya.patil3254#!/
[4]: https://twitter.com/Pradnya1208


[![github](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/c292abd3f9cc647a7edc0061193f1523e9c05e1f/icons/git.svg)][1]
[![linkedin](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/9f5c4a255972275ced549ea6e34ef35019166944/icons/iconmonstr-linkedin-5.svg)][2]
[![tableau](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/e257c5d6cf02f13072429935b0828525c601414f/icons/icons8-tableau-software%20(1).svg)][3]
[![twitter](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/c9f9c5dc4e24eff0143b3056708d24650cbccdde/icons/iconmonstr-twitter-5.svg)][4]

</div>

# <div align="center">Gesture based volume control</div>
<div align="center"><img src="https://github.com/Pradnya1208/Gesture-based-volume-control/blob/main/outputgif.gif" width="60%"></div>



## Overview:
In this project, we are going to learn how to use Gesture Control to change the volume of a computer. We first look into hand tracking and then we will use the hand landmarks to find gesture of our hand to change the volume. 


## Implementation:

**Libraries:**  `NumPy`  `pycaw` `pandas` `sklearn` `mediapipe` `cv2` `Matplotlib`

## Hands tracking:
The Mediapipe hand landmark model performs precise keypoint localization of 21 3D hand-knuckle coordinates inside the detected hand regions via regression, that is direct coordinate prediction. The model learns a consistent internal hand pose representation and is robust even to partially visible hands and self-occlusions.
<br>
<img src="https://github.com/Pradnya1208/AI-Virtual-Paint-App/blob/main/output/hands%20tracking.PNG?raw=true">
#### Code snippets:
```
class handDetector():
    def __init__(self, mode=False, maxHands=2, detectionCon=0.5, trackCon=0.5):
        self.mode = mode
        self.maxHands = maxHands
        self.detectionCon = detectionCon
        self.trackCon = trackCon
 
        self.mpHands = mp.solutions.hands
        self.hands = self.mpHands.Hands(self.mode, self.maxHands,
        self.detectionCon, self.trackCon)
        self.mpDraw = mp.solutions.drawing_utils
        self.tipIds = [4, 8, 12, 16, 20]
```
```
def findHands(self, img, draw=True):
    imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    self.results = self.hands.process(imgRGB)
    
    if self.results.multi_hand_landmarks:
        for handLms in self.results.multi_hand_landmarks:
            if draw:
                self.mpDraw.draw_landmarks(img, handLms,
                self.mpHands.HAND_CONNECTIONS)
 
    return img
```
```
cap = cv2.VideoCapture(1)
detector = handDetector()
while True:
    success, img = cap.read()
    img = detector.findHands(img)
```
We get all the hand landmarks with above code.
<img src="https://github.com/Pradnya1208/AI-Virtual-Paint-App/blob/main/output/landamrks.PNG?raw=true">
<br>
#### Getting the position values of hand landmraks:
```
def findPosition(self, img, handNo=0, draw=True):
    xList = []
    yList = []
    bbox = []
    self.lmList = []
    if self.results.multi_hand_landmarks:
        myHand = self.results.multi_hand_landmarks[handNo]
        for id, lm in enumerate(myHand.landmark):
            # print(id, lm)
            h, w, c = img.shape
            cx, cy = int(lm.x * w), int(lm.y * h)
            xList.append(cx)
            yList.append(cy)
            # print(id, cx, cy)
            self.lmList.append([id, cx, cy])
            if draw:
                cv2.circle(img, (cx, cy), 5, (255, 0, 255), cv2.FILLED)
 
    xmin, xmax = min(xList), max(xList)
    ymin, ymax = min(yList), max(yList)
    bbox = xmin, ymin, xmax, ymax
 
    if draw:
        cv2.rectangle(img, (xmin - 20, ymin - 20), (xmax + 20, ymax + 20),
        (0, 255, 0), 2)
 
    return self.lmList, bbox
```
`lmlist` in above function wll return all the landamrk positions.

## Volume control:
We need the landamrk positions of index figure and thumb for gesture control.
```
x1, y1 = lmList[4][1], lmList[4][2]
x2, y2 = lmList[8][1], lmList[8][2]
```
```
cx, cy = (x1 + x2) // 2, (y1 + y2) // 2
cv2.circle(img, (x1, y1), 15, (255, 0, 255), cv2.FILLED)
cv2.circle(img, (x2, y2), 15, (255, 0, 255), cv2.FILLED)
cv2.line(img, (x1, y1), (x2, y2), (255, 0, 255), 3)
cv2.circle(img, (cx, cy), 15, (255, 0, 255), cv2.FILLED)
```
Above code will create a circle and a connecting line for volume control.
#### Hypotenuse function for mapping the length to volume:
```
length = math.hypot(x2 - x1, y2 - y1)
```
#### Changing the volume of a computer:
We are going to use [pycaw](https://github.com/AndreMiras/pycaw).
<br>

**Usage:**
<br>
```
from ctypes import cast, POINTER
from comtypes import CLSCTX_ALL
from pycaw.pycaw import AudioUtilities, IAudioEndpointVolume
devices = AudioUtilities.GetSpeakers()
interface = devices.Activate(
    IAudioEndpointVolume._iid_, CLSCTX_ALL, None)
volume = cast(interface, POINTER(IAudioEndpointVolume))
volume.GetMute()
volume.GetMasterVolumeLevel()
volume.GetVolumeRange()
volume.SetMasterVolumeLevel(-20.0, None)
```
<br>

We are going to use methods `GetVolumeRange()` and `SetMasterVolumeLevel()`.
<br>

```
Hand range 50 - 300
Volume Range -65 - 0
```

#### Let us convert the hand range to our colume range:
```
vol = np.interp(length, [50, 300], [minVol, maxVol])
```
We will use `volume.SetMasterVolumeLevel(vol, None)` to set our volume.

#### Adding a volume bar and percentage:
```
volBar = np.interp(length, [50, 300], [400, 150])
volPer = np.interp(length, [50, 300], [0, 100])
cv2.rectangle(img, (50, 150), (85, 400), (255, 0, 0), 3)
cv2.rectangle(img, (50, int(volBar)), (85, 400), (255, 0, 0), cv2.FILLED)
cv2.putText(img, f'{int(volPer)} %', (40, 450), cv2.FONT_HERSHEY_COMPLEX, 1, (255, 0, 0), 3)
```







### Learnings:
`Computer vision`
`Landamark detection`
`Hands movement tracking using Mediapipe and cv2`







## References:
[mediapipe](https://mediapipe.dev/)
[hands tracking](https://google.github.io/mediapipe/solutions/hands.html)
[pycaw](https://github.com/AndreMiras/pycaw)
### Feedback

If you have any feedback, please reach out at pradnyapatil671@gmail.com


### 🚀 About Me
#### Hi, I'm Pradnya! 👋
I am an AI Enthusiast and  Data science & ML practitioner

[1]: https://github.com/Pradnya1208
[2]: https://www.linkedin.com/in/pradnya-patil-b049161ba/
[3]: https://public.tableau.com/app/profile/pradnya.patil3254#!/
[4]: https://twitter.com/Pradnya1208


[![github](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/c292abd3f9cc647a7edc0061193f1523e9c05e1f/icons/git.svg)][1]
[![linkedin](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/9f5c4a255972275ced549ea6e34ef35019166944/icons/iconmonstr-linkedin-5.svg)][2]
[![tableau](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/e257c5d6cf02f13072429935b0828525c601414f/icons/icons8-tableau-software%20(1).svg)][3]
[![twitter](https://raw.githubusercontent.com/Pradnya1208/Telecom-Customer-Churn-prediction/c9f9c5dc4e24eff0143b3056708d24650cbccdde/icons/iconmonstr-twitter-5.svg)][4]



