## Solution

```.py

class Raindrops:
    def __init__(self, n: int):
        self.n = n

    def pour(self, n: int) -> str:
        out = ""
        if n % 3 == 0:
            out += "Pling"
        if n % 5 == 0:
            out += "Plang"
        if n % 7 == 0:
            out += "Plong"
        if out == "":
            out = str(n)
        return out


test = Raindrops(30)
print(test.pour(30))

```


## Proof of work

<img width="1470" alt="Screenshot 2024-02-29 at 11 07 38" src="https://github.com/yuxuantaoisak/unit_3/assets/144768397/ecc22c1d-a39d-4978-a6f5-0dffeaf733b5">

