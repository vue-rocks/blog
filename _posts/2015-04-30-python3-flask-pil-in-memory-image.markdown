---
layout: post
title:  "Serving generated images from memory"
description: "Efficiently generating and serving images on-the-fly without the hassle of temporary files."
date:   2015-04-30 14:15:00+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
author: sander
---

In this post, we are going to implement a simple image placeholder service like [placehold.it][placeholdit]. Use-cases for disposable images extend beyond placeholder services, commonly used to encode text into an image in order to protect websites from being scraped for valuable information.

A simple approach could be to generate an image, write it to disc and then serve the static file. One could improve the solution by using temporary files, however the process still remains inefficient. The solution, of course, is to use in-memory files to cut out the process of writing the file to the hard drive.

We are using Pillow, a fork of Python's PIL (Python Imaging Library) that is actively maintained. While there is information out there on how to write an image into memory using StringIO module for Python2, using Python3 version io.StringIO results in an error:


```python
Traceback (most recent call last):
  File "image.py", line 10, in <module>
    image.save(string_io, 'PNG')
  File "~/.pyenv/versions/3.4.0/lib/python3.4/site-packages/PIL/Image.py", line 1685, in save
    save_handler(self, fp, filename)
  File "~/.pyenv/versions/3.4.0/lib/python3.4/site-packages/PIL/PngImagePlugin.py", line 631, in _save
    fp.write(_MAGIC)
TypeError: string argument expected, got 'bytes' 
```

As hinted in the error message, the solution is to use io.BytesIO instead. See a minimal working sample below:


```python
from io import BytesIO
from PIL import Image, ImageDraw

image = Image.new("RGB", (300, 50))
draw = ImageDraw.Draw(image)
draw.text((0, 0), "This text is drawn on image")

byte_io = BytesIO()
image.save(byte_io, 'PNG')
```

Putting it all together, let's implement a simple Flask service that takes one parameter, extracts dimensions, generates an image and writes the passed parameter as a text on the image. Flask is amazing, allowing us to write fully functional web service in just 30 lines of code:


```python
import re
from io import BytesIO
from flask import Flask, abort, send_file
from PIL import Image, ImageDraw

app = Flask(__name__)

@app.route('/<dimensions>')
def generate_image(dimensions):
    #Extract digits from request variable e.g 200x300
    sizes = [int(s) for s in re.findall(r'\d+', dimensions)]
    if len(sizes) != 2:
      abort(400)
    width = sizes[0]
    height = sizes[1]
    
    image = Image.new("RGB", (width, height))
    draw = ImageDraw.Draw(image)
    
    #Position text roughly in the center
    draw.text((width/2 - 25, height/2 - 5), dimensions)

    byte_io = BytesIO()
    image.save(byte_io, 'PNG')
    byte_io.seek(0)

    return send_file(byte_io, mimetype='image/png')
        
if __name__ == '__main__':
    app.run(debug=True)
 
```

To run this code, you may need to install few dependencies:


```bash
#Install dependencies
sudo apt-get install virtualenv python3.4-dev python-pip

#Create virtual environment
virtualenv -p /usr/bin/python3.4 venv

#Activate virtual environment
source venv/bin/activate

#Install libraries
pip install flask Pillow 

#Run server
python server.py

#http://localhost:5000/200x300
```

![Generated image][sample]

[placeholdit]: https://placehold.it/
[sample]: {{ site.baseurl }}/assets/python3-flask-pil-in-memory-image/sample.png
