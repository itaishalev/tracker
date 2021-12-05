# Tracker
# In this project I 
# # Press Shift+F10 to execute it or replace it with your code.
# Press Double Shift to search everywhere for classes, files, tool windows, actions, and settings.

import cv2
import keyboard
import numpy as np

# Create a VideoCapture object and read from input file
cap = cv2.VideoCapture('matkot.mp4')

# Check if video opened successfully
if (cap.isOpened() == False):
    print("Error opening video stream or file")

# Activate tracker
tracker = cv2.TrackerMOSSE_create()

# Read the first frame
ret, frame = cap.read()

# Select a boundary box
bbox = cv2.selectROI('Pendulum', frame, False)
tracker.init(frame, bbox)
# Initialize coordinates for later calculations of movement
prev_x = 0
prev_y = 0

# A function to draw a box around the tracked object
def drawbox(frame,bbox):
    x, y, w, h = int(bbox[0]),int(bbox[1]),int(bbox[2]),int(bbox[3])
    cv2.rectangle(frame, (x, y), (x+w, y+h), (255, 0, 0), 2, 1)

# Read until video is completed
while (cap.isOpened()):
# Capture frame-by-frame
    ret, frame = cap.read()
    ret, bbox = tracker.update(frame)  # Update the boundary box

    if ret == True:
        drawbox(frame, bbox)  # Draw the updated boundary box
        delta_x = int(bbox[0] - prev_x)  # calculate the x difference between current and previous frames
        delta_y = int(bbox[1] - prev_y)  # calculate the y difference between current and previous frames
        delta = delta_x, delta_y
        cv2.putText(frame, str(delta), (75, 50), cv2.FONT_ITALIC, 0.7, (255, 0, 0), 2)  # Type the difference between two frames

   # Display the resulting frame
        cv2.imshow('Pendulum', frame)

   # Press P on keyboard to pause the video
        if keyboard.is_pressed('p'):
            # Overlay the last two frames and display them
            dst = cv2.addWeighted(frame, 0.5, prev_frame, 0.7, 0)
            cv2.imshow('Before Adjusting', dst)

   # create new image of desired size and color (blue) for padding
            old_image_height, old_image_width, channels = prev_frame.shape
            new_image_width = old_image_width+10
            new_image_height = old_image_height+10
            color = (255, 0, 0)
            padded_result_prev = np.full((new_image_height, new_image_width, channels), color, dtype=np.uint8)
            padded_result_current = np.full((new_image_height, new_image_width, channels), color, dtype=np.uint8)

   # compute center offset
            x_center = (new_image_width - old_image_width) // 2
            y_center = (new_image_height - old_image_height) // 2

   # copy the previous frame into center of result image and shift it to adjust to the current position of the ball
            padded_result_prev[y_center+delta_y:y_center + old_image_height+delta_y, x_center+delta_x:x_center + old_image_width+delta_x] = prev_frame
            # copy the current frame into center of result image
            padded_result_current[y_center:y_center + old_image_height, x_center:x_center + old_image_width] = frame

   # Overlay the last two frames after shifting and display them
            dst = cv2.addWeighted(padded_result_current, 0.5, padded_result_prev, 0.7, 0)
            cv2.imshow('After Adjusting', dst)

            cv2.waitKey()  # wait until any key is pressed

   # Press Q on keyboard to exit
        if cv2.waitKey(30) & 0xFF == ord('q'):
            break  # Break the loop

   # Save the coordinates of the ball for next iteration
        prev_frame = frame
        prev_x = bbox[0] # save the x coordinate
        prev_y = bbox[1] # save the y coordinate

    else:
        break

# When everything done, release the video capture object
cap.release()

# Closes all the frames
cv2.destroyAllWindows()

