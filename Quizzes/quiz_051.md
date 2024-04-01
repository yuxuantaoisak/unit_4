## Solution

```.py

class Aircraft:
    def __init__(self, model, capacity):
        self.model = model
        self.capacity = capacity

    def get_info(self):
        return f"{self.model} (Capacity: {self.capacity})"


class Flight(Aircraft):
    def __init__(self, model, capacity, flight_num, origin, departure_time, destination):
        super().__init__(model, capacity)
        self.flight_num = flight_num
        self.origin = origin
        self.departure_time = departure_time
        self.destination = destination

    def get_info(self):
        return (f"Flight {self.flight_num} from {self.origin} to {self.destination}"
                f" departs at {self.departure_time}. Aircraft: {self.model} (Capacity: {self.capacity})")


test_1 = Aircraft("Boeing 737", 100)
test_2 = Flight("Boeing 737", 100,
                "MY132", "Kuala Lumpur",
                "10:00 AM", "Beijing")


def print_object_info(obj):
    return obj.get_info()


print(print_object_info(test_1))
print(print_object_info(test_2))


```


## Proof of work

<img width="1470" alt="Screenshot 2024-04-01 at 17 04 25" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/106901ce-0d30-45cf-a5fa-1d4973d9c2ef">


## UML Diagram

<img width="366" alt="Screenshot 2024-04-01 at 17 07 23" src="https://github.com/yuxuantaoisak/unit_4/assets/144768397/2b437724-57bf-441a-9993-223b9f1fd0aa">

