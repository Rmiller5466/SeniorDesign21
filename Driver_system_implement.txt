import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BOARD)

GPIO.setup(11, GPIO.OUT)
leftservo = GPIO.PWM(11, 50)
GPIO.setup(12, GPIO.OUT)
rightservo = GPIO.PWM(12, 50)

leftservo.start(0)
rigtservo.start(0)

time.sleep(2)

duty = 2

while duty <= 12:
	leftservo.ChangeDutyCycle(duty)
	rightservo.ChangeDutyCycle(duty)
	time.sleep(0.3)
	leftservo.ChangeDutyCycle(0)
	rightservo.ChangeDutyCycle(0)
	time.sleep()0.7
	duty = duty + 1

time.sleep(2)

leftservo.ChangeDutyCycle(7)
rightservo.ChangeDutyCycle(7)
time.sleep(2)

leftservo.ChangeDutyCycle(2)
rightservo.ChangeDutyCycle(2)
time.sleep(0.5)
leftservo.ChangeDutyCycle(0)
rightservo.ChangeDutyCycle(0)

leftservo.stop()
rightservo.stop()
GPIO.cleanup()
