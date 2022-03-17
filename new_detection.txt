# import required modules
from flask import Flask, render_template, Response 
import picamera 
import cv2
import socket 
import io

from flask import Flask, request, render_template, Response
import cv2
import RPi.GPIO as GPIO
import time

app = Flask(__name__) 
vc = cv2.VideoCapture(0)



@app.route('/') 
def index(): 
   """Video streaming .""" 
   return render_template('main.html')


GPIO.setmode(GPIO.BOARD) #Use Board numerotation mode
GPIO.setwarnings(False) #Disable warnings

# Use pin 12 for PWM signal

pwm_gpio = 12
frequence = 50
GPIO.setup(pwm_gpio, GPIO.OUT)
pwm = GPIO.PWM(pwm_gpio, frequence)
angle = 0

#thres = 0.45 # Threshold to detect object

classNames = []
classFile = "/home/pi/Object_Detection_Files/coco.names"
with open(classFile,"rt") as f:
    classNames = f.read().rstrip("\n").split("\n")

configPath = "/home/pi/Object_Detection_Files/ssd_mobilenet_v3_large_coco_2020_01_14.pbtxt"
weightsPath = "/home/pi/Object_Detection_Files/frozen_inference_graph.pb"

net = cv2.dnn_DetectionModel(weightsPath,configPath)
net.setInputSize(320,320)
net.setInputScale(1.0/ 127.5)
net.setInputMean((127.5, 127.5, 127.5))
net.setInputSwapRB(True)

#Set function to calculate percent from angle
def angle_to_percent (angle) :
    if angle > 180 or angle < 0 :
        return False

    start = 4
    end = 12.5
    ratio = (end - start)/180 #Calculate ratio from angle to percent

    angle_as_percent = angle * ratio

    return start + angle_as_percent

def getObjects(img, thres, nms, draw=True, objects=[]):
    classIds, confs, bbox = net.detect(img,confThreshold=thres,nmsThreshold=nms)
    #print(classIds,bbox)
    if len(objects) == 0: objects = classNames
    objectInfo =[]
    if len(classIds) != 0:
        for classId, confidence,box in zip(classIds.flatten(),confs.flatten(),bbox):
            className = classNames[classId - 1]
            if className in objects:
                objectInfo.append([box,className])
                if (draw):
                    cv2.rectangle(img,box,color=(0,255,0),thickness=2)
                    cv2.putText(img,classNames[classId-1].upper(),(box[0]+10,box[1]+30),
                    cv2.FONT_HERSHEY_COMPLEX,1,(0,255,0),2)
                    cv2.putText(img,str(round(confidence*100,2)),(box[0]+200,box[1]+30),
                    cv2.FONT_HERSHEY_COMPLEX,1,(0,255,0),2)

                 
                    #box[0], box[1] == x,y
                    line = 160
                    angle = 0
                    saturation = 30
                    if (box[0] < line - saturation):
                        angle = angle - 2
                    elif (box[0] > line+ saturation):
                        angle = angle + 2
    
                    #Init at 0°
                    pwm.start(0)
                    pwm.ChangeDutyCycle(angle_to_percent(angle))
                    time.sleep(0.025)


    return img,objectInfo


def gen(): 
   while True: 
       rval, frame = vc.read()
       result, objectInfo = getObjects(frame,0.45,0.2, objects=['cat'])
       cv2.imwrite('pic.jpg', result) 
       yield (b'--frame\r\n' 
              b'Content-Type: image/jpeg\r\n\r\n' + open('pic.jpg', 'rb').read() + b'\r\n')
       
@app.route('/video') 
def video(): 
   return Response(gen(), 
                   mimetype='multipart/x-mixed-replace; boundary=frame')


if __name__ == '__main__': 
    app.run(host='0.0.0.0', debug=False, threaded=True) 