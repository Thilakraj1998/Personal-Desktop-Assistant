# packages


```python
import pywhatkit as pwk
import pyttsx3 as tts
import datetime as dt
import wikipedia as wk
import pyjokes as pj
import PyPDF2 as pydf
import os
import subprocess
import docx 
import webbrowser as wb
import smtplib as sm
import pymongo as mg
import pandas as pd
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
```

# Activate Assistant


```python
cmd_taker()
```

    what is ur command: search python
    what is ur command: open youtube
    what is ur command: bye
    

# MongoDB connector


```python
# connection 
def get_dbConn():
    conn=mg.MongoClient("mongodb://localhost:27017/")
    mydb = conn["PyDB"]
    mycol=mydb["MailService"]
    return mycol
```

# fetching mailid from database


```python
def get_mailid(name):
    mycol=get_dbConn()
    df=pd.DataFrame.from_records(mycol.find_one({"name":name},{"_id":0,"name":1,"mailid":1,"num":0}))
    m=df.loc[0,'mailid']
    Mailstruct(m,name)
```

# adding new mail to Database


```python
def add_mail(uname,mail,num):
    mycol=get_dbConn()
    r=mycol.insert_one({"name":uname,"mailid":mail,"num":num})
    talk("new user added to database")
```

# fetching user phone number


```python
def get_contact(name):
    mycol=get_dbConn()
    df=pd.DataFrame.from_records(mycol.find({"name":name}))
    m=df.loc[0,"num"]
    send_msg(m)
    
```


```python
get_contact("vedant m")
```

    +919819740552
    testing random
    yes
    In 63 seconds web.whatsapp.com will open and after 20 seconds message will be delivered
    

# Mail Service


```python
def MailThem(id,sub,text):
    server = sm.SMTP_SSL('smtp.gmail.com', 465)
    server.login('TRDsAssistant@gmail.com', 'Br0KeN@1998')
    message = MIMEMultipart()
    body = text
    message['From'] = 'TRDsAssistant@gmail.com'
    message['To'] = id
    message['Subject'] = sub
    message.attach(MIMEText(body, 'plain'))
    server.send_message(message)
    talk("Mail has been send")
    server.close()
```

# Mail queries


```python
def Mailstruct(eid,name):
    talk("what is the subject?")
    sub=input("")
    talk("what is your message to "+name)
    body=input("")
    talk("Would you like to send it?")
    if(input()=="yes"):
        MailThem(eid,sub,body)
    else:
        talk("let me discard the mail!")
```

# voice engine


```python
def talk(text):
    engine = tts.init()
    voices = engine.getProperty('voices')
    rates=engine.getProperty('rate')
    engine.setProperty('rate',150)
    engine.setProperty('voice', voices[0].id)
    engine.say(text)
    engine.runAndWait()
    
```

# file reader


```python
#reading text file and return data
def read_file(file,style):
    text=""
    if style=="text file":
        cmd=file.replace("text file","")
        f = open(os.getcwd()+"\\"+cmd+".txt", "r")
        text= f.read()
    elif style=="word file":
        cmd=file.replace("word file","")
        doc = docx.Document(os.getcwd()+"\\"+cmd+".docx")
        all_paras = doc.paragraphs
        for para in all_paras:
            text+=para.text
    talk(text)

```

# external software exceuter


```python
def soft_opener(cmd):
    chromedir= "C:/Program Files (x86)/Google/Chrome/Application/chrome.exe %s"
    if "spotify" in cmd:
        talk("opening spotify for you")
        subprocess.Popen(['C:\\Users\\user\\AppData\\Roaming\\Spotify\\Spotify.exe'])
    elif 'word' in cmd:
        talk("opening Ms word for you")
        subprocess.Popen(['C:\\Program Files\\Microsoft Office\\Office15\\WINWORD.EXE'])
    elif 'chrome' in cmd:
        talk("opening chrome for you")
        subprocess.Popen(['C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe','-new-tab'])
    elif 'edge' in cmd:
        talk("opening Ms edge for you")
        subprocess.Popen(['C:\\Program Files (x86)\\Microsoft\Edge\\Application\\msedge.exe','-new-tab'])
    elif 'excel' in cmd:
        talk("opening Ms excel for you")
        subprocess.Popen(['C:\\Program Files\\Microsoft Office\\Office15\\EXCEL.EXE'])
    else:
        talk("no such software found in system")
```

# file reading command executer


```python
def file_type(file):
    if "text file"in file:
        file= file.replace("text file",'')
        read_file(file.strip(),"text file")
    elif "word file" in file:
        file= file.replace("word file",'')
        read_file(file.strip(),"word file")
    else:
        talk("no such file is managed by me")
```

# WhatsApp messeger


```python
def send_msg(contact):
    h=int(dt.datetime.now().strftime('%H'))
    m=int(dt.datetime.now().strftime('%M'))+2
    talk("what is the message?")
    msg=input()
    talk("should i send it?")
    if(input()=="yes"):
        pwk.sendwhatmsg(contact,msg,h,m)
        talk("message has been sent")
    else:
        talk("message discarded!")
```

# basic command executer


```python
def cmd_taker():
    g = dt.datetime.now().strftime('%H')
    g= int(g)
    if(g>12 and  g<=4):
        talk("good afternoon, how can i help you")
    elif(g<12 and g>0):
        talk("good morning, how can i help you")
    else:
        talk("good evening, how can i help you")
    chromedir= "C:/Program Files (x86)/Google/Chrome/Application/chrome.exe %s"
    st=True
    while(st==True):
        command = start_ass()
        if 'play' in command:
            song = command.replace('play', '')
            talk('playing ' + song)
            pwk.playonyt(song)
        elif 'time' in command:
            time = dt.datetime.now().strftime('%I:%M %p')
            talk('Current time is ' + time)
        elif 'wiki' in command:
            person = command.replace('wiki', '')
            info = wk.summary(person, 1)
            talk(info)
        elif 'date' in command:
            talk('sorry, I have a headache')
        elif 'joke' in command:
            talk(pj.get_joke())
        elif 'bye' in command:
            st=False
            talk("See you later")
        elif 'read' in command:
            file = command.replace('read', '')
            file_type(file)
        elif 'what are your features' in command:
            talk('I can tell jokes,play music for you,tell time,read text files for you and get details from wikipedia')
        elif ('thank' or 'thank you' or "thanks")in command:
            talk('happy to help you')
        elif "search" in command:
            cmd=command.replace("search","")
            url = "https://www.google.com.tr/search?q={}".format(cmd) 
            wb.get(chromedir).open(url)
        elif "youtube" in command:
            cmd=command.replace("youtube","")
            url="https://www.youtube.com/results?search_query={}".format(cmd)
            wb.get(chromedir).open(url)
        elif 'open' in command:
            cmd=command.replace("open ","")
            soft_opener(cmd)
        elif ('mail') in command:
            cmd=command.replace("mail","")
            cmd=cmd.strip()
            get_mailid(cmd)
        elif "add contact" in command:
            talk("what is the username")
            mid=input()
            talk("what is his Email id")
            em=input()
            talk("what is his contact number")
            nu=input()
            add_mail(mid,em,nu)
        elif "message" in command:
            cmd=command.replace("message","")
            get_contact(cmd.strip())
        else:
            talk('Please say the command again.')
    
```

# Command accepter


```python
def start_ass():
    cmd=input("what is ur command: ")
    cmd = cmd.lower()
    return cmd    
```
