# Criteria C: Development

## Existing tools

## List of techniques used


## Login system

my client required a login system for the application so that different users can have their unique profile pages and post the comments. I initially decided to use cookies
as a way of storing when a user is logged in. The code below shows my first attempt and I will explain in detail below. 

```.py
def index():
    if request.method == "POST":
        username = request.form.get('username')
        users = {'bob': 10, 'alice': 5}
        if username in users:
            session['current_user'] = users[username]
            return redirect(url_for('profile'))
    return render_template('index.html')

```

This code is run when the request from the client received by the server is of type POST `if request.method == "POST"`. This happens when the user clicks on the login button
on the index.html.page. Then, I proceed to get the variables from the login form including the name and password, this is contained in the dictionary request. After checking
the database with the SQL query... Then I set the cookie for the user with the code `response.set_cookie('user_id', f"{user_id}")`, note that the cookie is like a
dictionary with key and values both strings. I used a f string to convert the id which is an integer to a string. 


However, based on my research about cookies and testing in the browser, I found out that the cookie is not a secure way of storing the user information since this is variable
stored in the browser on the client side, which can be easily changed, causing identity thief if not hashed. So my solution to this issue was to change from client to
server side by using sessions. 

The code below shows that I could store the current user in the dictionary session. "xxx" [ref 1]

```.py

session['current_user'] = users[username]
return redirect(url_for('profile'))

```

In order to use the session variable, I needed to define an initial secret in the variable of the Flast application. I did this in the code below and generated a random
string as secure key. 


```.py

app = Flask(__name__)
app.secret_key = "randomtextwith12345"

```

Route table

| Route/endpoint           | Methods   | Description                                                                                    |
|--------------------------|-----------|------------------------------------------------------------------------------------------------|
| /login                   | GET, POST | This endpoint allows the user to login into the application. User is stored in the app session |
| /signup                  | GET, POST |                                                                                                |
| /mainpage                | GET, POST | A search bar                                                                                   |
| /profile/<int: user_id>  | GET, POST | Put method to allow editing profile                                                            |
| /profile/edit            | GET, POST |                                                                                                |
| /category/<str: cat>     | GET, POST | Put string as input(i.e, anime, suspense)                                                      |
| /category/<str: cat>/add | GET, POST |                                                                                                |
| /search                  | GET       |                                                                                                |
| /admin                   |           | Endpoint for the admin                                                                         |

