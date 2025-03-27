# MakeyBot.py
from gpiozero import PWMLED, LED, AngularServo
from time import sleep
import time

# Debug Settings
debug_messages = True  
print("Debug Messages are 'ON'" if debug_messages else "Debug Messages are 'OFF'")

# Define Traffic Lights
red_led = LED(5)
yellow_led = LED(6)
green_led = LED(13)

# Servo Configuration (Refined for Less Jitter)
maxCorrection = 0.20  # Reduced to prevent extreme correction values
minCorrection = 0.20  
maxPW = (2.0 + maxCorrection) / 1000
minPW = (1.0 - minCorrection) / 1000

servo = AngularServo(12, min_pulse_width=minPW, max_pulse_width=maxPW)

# Raspberry Pi Pins - Left Eye
left_red_pwm = PWMLED(4)
left_green_pwm = PWMLED(18)
left_blue_pwm = PWMLED(23)

# Raspberry Pi Pins - Right Eye
right_red_pwm = PWMLED(17)
right_green_pwm = PWMLED(27)
right_blue_pwm = PWMLED(22)

def arm(arm_position, move_count):
    """Moves the servo arm back and forth smoothly based on user input."""
    for _ in range(move_count):
        servo.angle = arm_position  # Move to the set position
        sleep(1)  
        servo.angle = 0  # Return to default position
        sleep(1)

def Get_Robot_Data(left_eye_status, right_eye_status):
    """Controls the RGB values of both eyes."""
    if debug_messages:
        print("Running Get_Robot_Data function")
        print(f"Left Eye: {left_eye_status}, Right Eye: {right_eye_status}")

    # Apply RGB values with safety clipping
    left_red_pwm.value = max(0, min(1, left_eye_status['red_RGBLED']))
    left_green_pwm.value = max(0, min(1, left_eye_status['green_RGBLED']))
    left_blue_pwm.value = max(0, min(1, left_eye_status['blue_RGBLED']))

    right_red_pwm.value = max(0, min(1, right_eye_status['red_RGBLED']))
    right_green_pwm.value = max(0, min(1, right_eye_status['green_RGBLED']))
    right_blue_pwm.value = max(0, min(1, right_eye_status['blue_RGBLED']))

def Traffic_Lights(red_time, yellow_time, green_time):
    """Controls the three-sequence LED system. Runs only once instead of looping forever."""
    if debug_messages:
        print("Running Traffic_Lights function")
    
    red_led.on()
    sleep(red_time)
    red_led.off()

    yellow_led.on()
    sleep(yellow_time)
    yellow_led.off()

    green_led.on()
    sleep(green_time)
    green_led.off()

def get_valid_float(prompt, min_val, max_val):
    """Safely gets a valid float within a range."""
    while True:
        try:
            value = float(input(prompt))
            if min_val <= value <= max_val:
                return value
            print(f"Error: Enter a value between {min_val} and {max_val}.")
        except ValueError:
            print("Invalid input! Please enter a valid number.")

def get_valid_int(prompt, min_val):
    """Safely gets a valid integer above a minimum value."""
    while True:
        try:
            value = int(input(prompt))
            if value >= min_val:
                return value
            print(f"Error: Enter a number greater than or equal to {min_val}.")
        except ValueError:
            print("Invalid input! Please enter a valid whole number.")

def main():
    print("Welcome To The STEAM Clown Makey Bot")

    while True:
        try:
            # Get user input for left eye
            left_eye_status = {
                'red_RGBLED': get_valid_float("Enter Red value for Left Eye (0-1): ", 0, 1),
                'green_RGBLED': get_valid_float("Enter Green value for Left Eye (0-1): ", 0, 1),
                'blue_RGBLED': get_valid_float("Enter Blue value for Left Eye (0-1): ", 0, 1)
            }

            # Get user input for right eye
            right_eye_status = {
                'red_RGBLED': get_valid_float("Enter Red value for Right Eye (0-1): ", 0, 1),
                'green_RGBLED': get_valid_float("Enter Green value for Right Eye (0-1): ", 0, 1),
                'blue_RGBLED': get_valid_float("Enter Blue value for Right Eye (0-1): ", 0, 1)
            }

            # Get servo movement input
            arm_position = get_valid_float("Enter servo arm position (0-90 degrees): ", 0, 90)
            move_count = get_valid_int("Enter how many times the arm should move: ", 1)

            # Get traffic light durations
            red_time = get_valid_float("Enter duration for Red LED (seconds): ", 0.1, 10)
            yellow_time = get_valid_float("Enter duration for Yellow LED (seconds): ", 0.1, 10)
            green_time = get_valid_float("Enter duration for Green LED (seconds): ", 0.1, 10)

            # Apply eye colors
            if debug_messages:
                print("Calling Get_Robot_Data function with:", left_eye_status, right_eye_status)
            Get_Robot_Data(left_eye_status, right_eye_status)

            # Start traffic lights
            if debug_messages:
                print("Starting traffic light sequence with times:", red_time, yellow_time, green_time)
            Traffic_Lights(red_time, yellow_time, green_time)

            # Move the servo arm
            if debug_messages:
                print(f"Moving servo arm {move_count} times to position {arm_position}")
            arm(arm_position, move_count)

        except KeyboardInterrupt:
            print("\nProgram stopped by user.")
            break  # Exit loop on Ctrl+C

        time.sleep(0.5)  # Small delay before next input loop

# Run the program
main()


