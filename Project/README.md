# Unit 4: A better reddit


# Criteria C: Development

## Existing tools

## List of techniques used


## Login system

A login system for the application is required so that different users can have their unique profile pages and post the comments. I initially decided to use cookies
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
on the `index.html` page. Then, I proceed to get the variables from the login form including the name and password, this is contained in the dictionary request. After checking
the database with the SQL query... Then I set the cookie for the user with the code `response.set_cookie('user_id', f"{user_id}")`. Note that the cookie is like a
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

However, after doing some research, I realized that simply assigning user IDs to sessions is not a secure practice. A user can easily access the browser's inspector and change the user ID, thus gaining unauthorized access to other users' accounts. 

In order to solve this problem, I decided to use JWT token. Each sebsequent request after the user is logged in will include the JWT, allowing the user to access the routes that are permitted. A user ID encoded with JWT looks like this:

```.py
jwt.encode({'user_id': user_id}, token_encryption_key, algorithm='HS256')`.

```

With this feature, the login system is safe and free of data leaks so that personal information is protected. 


To ensure that some parts of the website are only accessible for authorized users, the login status of the user needs to be confirmed in some routes. In order to validate the login status of the current user, I designed a user validation function called `login_needed`. I will explain how it works in detail. 


```.py

def login_needed(f):

    @wraps(f)
    def decorated(*args, **kwargs):
        if 'token' not in session:
            return redirect(url_for('login'))
        try:
            data = jwt.decode(session['token'], token_key, algorithms=['HS256'])
        except:
            return redirect(url_for('login'))

        # If the token is valid, continue to the original function with the original arguments
        return f(*args, **kwargs)

    # Return the new decorated function
    return decorated

```

First, the `@wraps` function is used to preserve the metadata of the original function, such as the name and docstring. Then, the next lines check if 'token' key exists in session. If not, the user is redirected to login page. Next, a `try` block attempts to decode the token using `jwt.decode()`. If the token is valid, it will allow the user to proceed within the website that requires authorization, if the token is invalid, expired, or tampered with, the user is redirected to the login page which is achieved by the use of `except` block. The try/except block can test the first statement, and provides a solution if the first one doesn't work. This solution effectively addresses the authorization issue and enhances the security of the web application.


## Edit/create comments

As part of the requirements of the project, the user should be able to create/edit their comments. 

```.py

create_comment = f"INSERT INTO comments (content, datetime, user_id, post_id) VALUES ('{comment}', '{datetime.now()}', '{user_id['user_id']}', '{post_id}')"
db.run_save(query=create_comment)

```
This is a snippet of codes from the function responsible for creating comments. An SQL query is used to add the content, time, user id, and post id to the table `comments`, and they are saved into the database by the `run_save` function. 


The function of editing existing comments is also required. Initially, I tried to use the same SQL query(insert into) to fulfil this. However, I found out that this would create repeating comments instead of updating the existing ones as required by the user. After researching about SQL queries, I decided to use different SQL statement to achieve this function. Below shows my attempt.

```.py

if request.method == 'POST':
    content = request.form['content']

    query = f"UPDATE comments SET content='{content}'"
    db.run_save(query=query)
    return redirect(url_for('profile'))

```

First, the code checks if the condition is met(if the request type is post, meaning creating or updating a resource). Then, the content of the comment is retrieved by `request.form` dicitonary from the user, and is defined as `content` variable. For the following SQL query, the new value is updated with `update` without creating repeated value, and `set` sets new values into existing values in the `comments` table. This SQL query updates existing comments without creating redundant and repetitive comments. 

## Uploading images

It was one of the most challenging parts to make my website allow users to upload images. 

Since the images need to be uploaded but also fetched when user is viewing the post, I broke down this problem into 2 steps: uploading the image and fetching it. This shows my computational thinking of decomposition. In the process of uploading images, the name of the image needs to be stored into a folder. I achieved this by the codes below.

```.py

if 'image' in request.files:
    image = request.files['image']
    if image.filename != '':
        filename = secure_filename(image.filename)
        image.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
    else:
        filename = None

```

This code first checks if the user has submitted an image. Then, the `secure_filename` function is used to sanitize the filename. Otherwise, an insecure filename such as "/xxx/xxx/pswd" will lead to the file being saved in an unintended directory[ref], compromising the website's security and making it prone to malicious activities. Then, the code saves the image into the upload folder. The `os.path.join` concatenates filename and directory, ensuring that correct path separators are used. If the user didn't upload image, the code sets the variable to `None`. Each step of this process shows my algorithmic thinking abilities. 

To fetch the image, I designed an `if` condition in my html side. The code is shown below. 

```.html

{% if post[5] != 'None' %}
            <img src="/static/photos/{{ post[5] }}" class="img-fluid" alt="Responsive image" style="height: auto; width: 400px;">
{% endif %}

```

I first fetched the filename if it exists, then put it into the corresponding directory (/static/photos). This ensures that the correct image is fetched from the relevant directory and database. This shows my ability to think abstractly. 


## Following

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


# Criteria D: Functionalities

A video demonstrating the testing for the success criteria

# Criteria E: Evaluation

A table of functionality tested. 


# Citations

https://jwt.io/introduction

https://medium.com/@sujathamudadla1213/what-is-the-use-of-secure-filename-in-flask-9eef4c71503b

