---
layout: post
title:  "Python logging handler for Mailgun"
description: "Sending application exception notifications via Mailgun."
date:   2015-10-12 13:25:30+02:00
size: 12
sizeDesktop: 4
textColor: grey-50
metaColor: grey-600
comments: true
author: sander
tags: [mailgun, python, flask, log, back-end]
---

Yet another reason to love python is the wonderful logging API provided by a standard library. Having a decent logger provided by the language saves developers time from integrating with some third-party library, but most importantly it contributes to a clean system where all modules use the same logger. Out of the box, there is a [handler][logging] for almost any typical use-case, but the best feature is the option to create your own with relative ease. So-far we've been using [SMTPHandler][smtp-handler], but since moving to [Mailgun][mailgun], I decided to experiment with a custom handler to submit logs via their API instead. I'd like to mention that creating a custom logging handler is not a requirement, it is still possible to use their SMTP server with SMTP handler. To get started, we need to extend Handler baseclass and override emit method to make a POST request.


```python
import logging
import requests

class MailgunHandler(logging.Handler):

    def __init__(self, api_url, api_key, sender, recipients, subject):
        # run the regular Handler __init__
        logging.Handler.__init__(self)
        self.api_url = api_url
        self.api_key = api_key
        self.sender = sender
        self.recipients = recipients
        self.subject = subject

    def emit(self, record):
        # record.message is the log message
        for recipient in self.recipients:
            data = {
                "from": self.sender,
                "to": recipient,
                "subject": self.subject,
                "text": self.format(record)
            }
            requests.post(self.api_url, auth=("api", self.api_key), data=data)
```

I have created a simple flask application that registers the handler and also implemented a route that triggers an error to test it out.


```python
import logging
from flask import Flask
from mailgun_handler import MailgunHandler

app = Flask(__name__)
# Setup logging handler
mail_handler = MailgunHandler(
    api_url='https://api.mailgun.net/v3/fadeit.dk/messages',
    api_key='secret',
    sender='noreply@fadeit.dk',
    recipients=['ss@fadeit.dk', 'ja@fadeit.dk'],
    subject='Application error!'
)
mail_handler.setLevel(logging.ERROR)
mail_handler.setFormatter(logging.Formatter('''
    Message type:       %(levelname)s
    Location:           %(pathname)s:%(lineno)d
    Module:             %(module)s
    Function:           %(funcName)s
    Time:               %(asctime)s
    
    Message:
    
    %(message)s
'''))
app.logger.addHandler(mail_handler)

@app.route('/error')
def error():
    raise Exception("Beds are burning!")
        
if __name__ == '__main__':
    #app.run(debug=True) #Handler isn't executed if app is run in debug mode
    app.run()
```


[logging]: https://docs.python.org/3/library/logging.handlers.html
[mailgun]: https://www.mailgun.com/
[smtp-handler]: https://docs.python.org/3/library/logging.handlers.html#smtphandler
