# omada6EnsSyst5-30
#line follower robot ομάδα 6, 5:30-7
from machine import Pin, PWM
from time import sleep

# Sensor pins (1 = μαύρο, 0 = άσπρο)
sensor_left = Pin(2, Pin.IN)
sensor_right = Pin(26, Pin.IN)

# Motor PWM control pins
IN1 = PWM(Pin(11))  # Right motor forward
IN2 = PWM(Pin(10))  # Right motor backward (unused)
IN3 = PWM(Pin(9))   # Left motor forward
IN4 = PWM(Pin(8))   # Left motor backward (unused)

# Set PWM frequency
IN1.freq(1000)
IN2.freq(1000)
IN3.freq(1000)
IN4.freq(1000)

# Speed settings
SPEED = 40000       # πιο αργό για ακρίβεια
TURN_SPEED = 0      # ένας τροχός σταματημένος για "κλειστή" στροφή

# --- Κινήσεις ---
def forward():
    IN1.duty_u16(SPEED)
    IN2.duty_u16(0)
    IN3.duty_u16(SPEED)
    IN4.duty_u16(0)

def hard_left():
    IN1.duty_u16(SPEED)  # δεξιός τροχός κινείται
    IN2.duty_u16(0)
    IN3.duty_u16(0)      # αριστερός σταματημένος
    IN4.duty_u16(0)

def hard_right():
    IN1.duty_u16(0)      # δεξιός σταματημένος
    IN2.duty_u16(0)
    IN3.duty_u16(SPEED)  # αριστερός τροχός κινείται
    IN4.duty_u16(0)

def stop():
    IN1.duty_u16(0)
    IN2.duty_u16(0)
    IN3.duty_u16(0)
    IN4.duty_u16(0)

# --- Κατάσταση στάσης (μόνο στη RAM) ---
permanent_stop = False

# --- Κεντρική Λούπα ---
while True:
    if permanent_stop:
        stop()
        continue

    left = sensor_left.value()
    right = sensor_right.value()

    print("Left:", left, "Right:", right)

    # Μόνιμο stop αν και οι δύο δουν μαύρο
    if left == 1 and right == 1:
        print("STOP: Και οι δύο αισθητήρες είδαν μαύρο. Τερματισμός.")
        stop()
        permanent_stop = True

    # Αν δει μαύρο μόνο δεξιά → στροφή αριστερά
    elif left == 0 and right == 1:
        hard_left()

    # Αν δει μαύρο μόνο αριστερά → στροφή δεξιά
    elif left == 1 and right == 0:
        hard_right()

    # Αν δουν και οι δύο άσπρο → ευθεία
    elif left == 0 and right == 0:
        forward()

    # Μικρή καθυστέρηση για σταθερότητα
    sleep(0.01)
