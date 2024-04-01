## Solution

```.py

from cmath import pi


class Wheel:
    def __init__(self, size):
        self.size = size

    def get_size(self):
        return self.size

    def get_perimeter(self):
        return 2 * pi * self.size

    def get_km_per_rotation(self):
        perimeter_in_km = self.get_perimeter() / 39370
        return perimeter_in_km


class Bicycle:
    def __init__(self, wheel_size, frame_material):
        self.wheel = Wheel(wheel_size)
        self.frame_material = frame_material

    def ride(self):
        print(f"Bicycle Information: Wheel Size - {self.wheel.get_size()}, Frame Material - {self.frame_material}")


my_bicycle = Bicycle(26, "Aluminum")
wheel = Wheel(26)
print(wheel.get_km_per_rotation())


```


## Proof of work


<img width="1470" alt="Screenshot 2024-04-01 at 17 28 20" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/d3b3e87a-cbc0-445b-9d48-98be23721ee6">


## UML Diagram

<img width="359" alt="Screenshot 2024-04-01 at 17 25 57" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/47ae8ded-866d-4a56-ae04-ef522a3f7b43">
