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

<img width="1470" alt="Screenshot 2024-04-01 at 15 21 28" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/62191aa7-0979-414b-850d-be5ca8e2e3d2">

