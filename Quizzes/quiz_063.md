A medical centre uses a computer system to manage both patients’ data and appointments. 

This system, which is used by the doctors, 
nurses and secretaries, has two unordered files: a patient's file and an appointment file.

Write the pseudocode that the processing must follow when the system sends out the text reminders.

```.py

read 'appointments'

for every value in appointment:
  if appointment['date'] == tomorrow:
    id = appointment['id']
    for every value in patient:
      define item
      if item['id'] == id:
        message = "You have an appointment tomorrow"
        send message to patient['phone']

```

