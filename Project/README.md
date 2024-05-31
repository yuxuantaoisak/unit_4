# Unit 4: A food review web application: MenuLog


# Criteria C: Development

## Existing tools

1. SQL queries
2. Pycharm
3. Relational databases
4. Python
5. Chrome/safari

## List of techniques used

1. Sanitizing file
2. Get/post methods
3. Sessions
4. Interacting with databases
5. For loops
6. If statements
7. OOP
8. Token based authentification
9. CSS styling




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
the database with a SQL query, I set the cookie for the user with the code `response.set_cookie('user_id', f"{user_id}")`. Note that the cookie is like a
dictionary with key and values both strings. I used a f string to convert the id which is an integer to a string. 


However, based on my research about cookies and testing in the browser, I found out that the cookie is not a secure way of storing the user information since this is variable
stored in the browser on the client side, which can be easily changed, causing identity thief if not hashed. So my solution to this issue was to change from client to
server side by using sessions. 

The code below shows that I could store the current user in the dictionary session.

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

In order to solve this problem, I decided to use JWT token. Each sebsequent request after the user is logged in will include the JWT, allowing the user to access the routes that are permitted[ref 1]. A user ID encoded with JWT looks like this:

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

This code first checks if the user has submitted an image. Then, the `secure_filename` function is used to sanitize the filename. Otherwise, an insecure filename such as "/xxx/xxx/pswd" will lead to the file being saved in an unintended directory[ref 2], compromising the website's security and making it prone to malicious activities. Then, the code saves the image into the upload folder. The `os.path.join` concatenates filename and directory, ensuring that correct path separators are used. If the user didn't upload image, the code sets the variable to `None`. Each step of this process shows my algorithmic thinking abilities. 

To fetch the image, I designed an `if` condition in my html side. The code is shown below. 

```.html

{% if post[5] != 'None' %}
            <img src="/static/photos/{{ post[5] }}" class="img-fluid" alt="Responsive image" style="height: auto; width: 400px;">
{% endif %}

```
Here, `post` is a list with several elements, including title, author, image, etc. `post[5]` refers to the sixth element which is the image. 

I first fetched the filename if it exists, then put it into the corresponding directory (/static/photos). This ensures that the correct image is fetched from the relevant directory and database. This shows my ability to think abstractly. 


## Liking/unliking posts

As part of the project's requirements, liking/unliking posts is a key feature to this web application. Initially, I used a very simple SQL query to achieve this feature. 


```.py

like_query = f"INSERT INTO likes (user_id, post_id) VALUES ('{user_id}', '{post_id}')"
db.run_save(query=like_query)
```

The `like_query` interacts with the database `likes`, inserting two values, `user_id` and `post_id` into the database. However, this code would allow the user to like the same post for multiple times, which I wanted to avoid. The user should only be able to like the post once, then the like button should turn to unlike button. 

In order to achieve this, I defined a variable `is_liked`. It checks if the post is already liked by the user or not. With this variable, I am now able to restrict the user from liking multiple times. I will explain how it works in detail. 


```.py

check_like_query = f"SELECT * FROM likes WHERE user_id = {user_id} AND post_id = {post_id}"
is_liked = db.run_fetchone(query=check_like_query)

if is_liked:
        # If the like record exists, delete it (unlike)
        unlike_query = f"DELETE FROM likes WHERE user_id = {user_id} AND post_id = {post_id}"
        db.run_save(query=unlike_query)

else:
        # If the like record doesn't exist, create it (like)
        like_query = f"INSERT INTO likes (user_id, post_id) VALUES ('{user_id}', '{post_id}')"
        db.run_save(query=like_query)

```

The `is_liked` function checks if a like record already exists in the database `likes` which is responsible for all like records of all users and posts. If `is_liked` value is not empty, the query will delete the past like record. Otherwise, a like record will be added to the database. 

Ideally, the button should change from "like" to "unlike" in the process of liking the post, preventing the user from liking twice. To achieve this, I incorporated python code from app.py file into the html file. I will explain below. 


```.html

{% if is_liked %}
    <form action="{{ url_for('like_post', post_id=post[0]) }}" method="POST">
        <button type="submit" class="btn btn-primary me-2">
            <i class="bi bi-heart-fill"></i> Unlike
        </button>
    </form>
{% else %}
    <form action="{{ url_for('like_post', post_id=post[0]) }}" method="POST">
        <button type="submit" class="btn btn-outline-primary me-2">
            <i class="bi bi-heart"></i> Like
        </button>
    </form>
{% endif %}

```
Just like how the python code works, the statement `is_liked` from the python file checks if the post is liked or not. If yes, an unfollow button is generated using the line `<i class="bi bi-heart-fill"></i> Unlike`. Otherwise, a like button is generated with the line `<i class="bi bi-heart"></i> Like`. During the thought process, I showed pattern recognition as a computational thinking skill where I realized that the python variable can be used in html file to create a button that changes between "like" and "unlike". 



# Criteria D: Functionalities

https://youtu.be/UgSG4TuRcic


# Criteria E: Evaluation

## Evaluation by alpha tester

The tester is very impressed with the sorting tabs on top, which has a navigation and sorting function that the tester found very useful. Moreover, the tester likes the following tab where they can check the followed posts. However, they suggested that the visual design of the rating of posts that are categorized as [restaurant] can be more appealing and stood-out, enhancing the user experience. Also, they would love to see the total likes of a post. Also, the tester highlighted that they need to go to profile page every time to delete or edit a comment/post, which makes it a complicated process especially for those who intend to edit their posts often. This feedback is very valuable for the future development of this project. 

## Evaluation by beta tester

The tester appreciates the function of sorting posts by time and likes. However, they suggested that the navigation can be done easier by just clicking the category line, such as [recipe], on the post, instead of going to the navigation tab every time. They suggested that this can be done simply by using a "GET" method, which can make the text a hyperlink to the respective category. Moreover, the tester mentioned that the sorting function on the main page can be further improved. It would be good if there are options such as "oldest" or "least liked" which can reflect the history of discussions and posts that are underappreciated. The tester said that this can be done by interacting with the database, just like other existing sorting functions. This feedback is very valuable for the future development of this project. 

See transcript for interview in appendix

# Citations

1. "JWT Introduction", Accessed 31 May, 2024. https://jwt.io/introduction

2. "What is the use of secure_filename if flask", Accessed 31 May, 2024. https://medium.com/@sujathamudadla1213/what-is-the-use-of-secure-filename-in-flask-9eef4c71503b


# Appendix

## Evidence of consultation



Interview transcript from alpha tester: 

I recently got to test the new app, and I’ve got to say, I’m really impressed with a lot of its features. The sorting tabs at the top are awesome—they make navigating and finding what I need super easy. The "following" tab is another great feature. It’s so handy to have a spot where I can quickly check the posts I’m following.

But I do have a couple of suggestions. The rating design for [restaurant] posts could be a bit more eye-catching. It would make the app look cooler and improve the user experience. Also, showing the total likes on a post more prominently would be great. It gives a better idea of what’s popular and could boost engagement.

One thing that’s a bit of a hassle is having to go to my profile page to delete or edit a comment or post. It’s a bit clunky, especially for frequent edits. It would be awesome if we could do that directly from the post or comment itself.

Overall, the app is really promising, and I’m excited to see these improvements in future updates!










Interview transcript from beta tester:

I recently tested the new app, and I appreciate several of its features, especially the ability to sort posts by time and likes. However, I have a few suggestions for improvement.

Firstly, navigation could be simplified. Instead of going to the navigation tab every time, it would be much easier if clicking on the category line (e.g., [recipe]) on a post would take you directly to that category. This can be implemented using a simple "GET" method to make the category text a hyperlink to the respective category page.

Additionally, the sorting function on the main page could be enhanced. It would be beneficial to include options like "oldest" or "least liked" to better reflect the history of discussions and highlight posts that may be underappreciated. This can be achieved by interacting with the database similarly to how the existing sorting functions are implemented.

Overall, these enhancements would make the app more user-friendly and efficient. This feedback is crucial for the future development of the project, and I'm looking forward to seeing these improvements in future updates.


