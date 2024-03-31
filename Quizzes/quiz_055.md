## Solution

```.py

from math import sqrt


class Darts:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def calculate(self):
        distance = sqrt(self.x**2 + self.y**2)
        if distance <= 1:
            pts = 10
        elif distance <= 5:
            pts = 5
        elif distance <= 10:
            pts = 1
        else:
            pts = 0
        return pts


test = Darts(3, 5).calculate()
print(test)

```

## Proof of work

![Screenshot 2024-03-04 at 20 46 55](https://github.com/yuxuantaoisak/unit_3/assets/144768397/ad07d52d-86cf-45f1-a140-c024fda1a77e)
