# How to Make a Backend Server using Flask and Docker
You can use any editor you want, but for this tutorial I will be using VSCode.

## Step 1: initializing the project
First, we're going to initialize a new node project.

Create a new folder called MyServer and open it in VSCode.

Open a terminal and navigate to the folder.
* If you're using VSCode, you can open a new terminal in the "Terminal" menu in the title bar, or with the shortcut Ctrl+Shift+`.
* If you're using Webstorm or Intellij, you can open a new terminal by clicking the "Terminal" menu on the bottom bar or with the shortcut Alt+f12.
* If you're using a plain text editor, you can open a new cmd and typing `cd [path-to-your-folder]`.

## Step 2: install Flask
Flask is a web app framework written in python, and is used to quickly make APIs in python.

To install it, run the command `pip install Flask`

Then, run the command `pip freeze > requirements.txt` to create a pip requirements file.

## Step 3: create dockerfile
Create a file called Dockerfile. In this file, we will put the instructions for building a docker image out of our server. Add the following code to the Dockerfile:
```dockerfile
FROM python:3.11-alpine

WORKDIR /app
COPY ./requirements.txt /app/requirements.txt
RUN pip install -r requirements.txt

COPY . .
ENTRYPOINT ["python"]
CMD ["server.py"]
```

## Step 4: create the server file
Create a file called `server.py`. This is the file in which we will put our API methods.

By now, your file structure should look like this:
```
├── MyServer
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── server.py
```

In `server.py`, add the following code:
```py
from flask import Flask

app = Flask(__name__)
```

This will first import the Flask package, then initialize a new flask app.

Next, we will add an index app route. To do this, add the following code under what we just added:
```py
@app.route('/', methods=['GET'])
def index():
    return 'Hello World!'
```
This will listen to GET requests to `/` and respond with "Hello World!".

Finally, we will start the server by adding the following code:
```py
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000, debug=True)
```
## Step 5: create and run the docker image
Run the command `docker build . -t myserver`. This will create the docker image.

To run the image, use the command `docker run -it --rm -p 3000:3000 myserver`

Finally, navigate to `http://localhost:3000` and you should be met with "Hello World!".

To stop the container, use the shortcut Ctrl+c in the terminal window.

## Step 6: add more endpoints
You can now add more endpoints similarly to how we created the index endpoint.