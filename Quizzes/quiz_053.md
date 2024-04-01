## Solution

```.py

class PalNum:
    def __init__(self, a, b):
        self.a = a
        self.b = b

    def get_pal_list(self):
        out = []
        for i in range(self.a, self.b + 1):
            temp_len = len(str(i))
            ex = ""
            ex_2 = ""
            for n in range(temp_len):
                ex += str(i)[n]
                for m in range(temp_len, -1, -1):
                    ex_2 += str(i)[m-1]
                    if ex_2 == ex:
                        out.append(ex)
                        break  # Break the loop once a palindrome is found
        return out


test_1 = PalNum(10, 100)
print(test_1.get_pal_list())

```

## Proof of work

<img width="1470" alt="Screenshot 2024-04-01 at 15 39 33" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/1079fe10-5e34-419a-b408-e6923f9c61a6">


## UML Diagram

<img width="419" alt="Screenshot 2024-04-01 at 15 41 13" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/a8564155-4f21-4f76-9055-629196657f9b">
