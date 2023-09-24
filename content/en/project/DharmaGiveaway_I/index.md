---
title: Hey browie! What if we do a Giveaway?
summary: Web Scraping an Instagram Giveaway
tags:
- Data Science
- Web Scraping
- python
date: "2023-09-24T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Merkaba Coaster Set, made by BlackPhillipDesign
  focal_point: Smart

links:
- icon: instagram
  icon_pack: fab
  name: Follow
  url: https://www.instagram.com/blackphillipdesign/
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""
# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---


Amidst a lively coffee session with [Sonya](https://www.instagram.com/sonya_xz/), ideas converged, worlds collided, leading to the conclusion that a collaboration between [BlackPhillip](https://www.instagram.com/blackphillipdesign/) (my product design business) and [Dharma](https://www.instagram.com/dharmainkstudio/) (his tattoo studio) was imminent. As the second pivotal point of this spontaneous meeting, we came to agree that the first instance of this official collaboration series should be a 'Giveaway.'

### The Challenge of the Giveaway

Almost instantly, popping inside my head, much like a flea on a stray dog: How would participant counts be managed, and what actions would be required of them to filter data based on these parameters? It's common knowledge that activities of this nature often involve the engagement of a community of followers in certain actions to qualify as participants and at the same time to boost the profiles pages involved in organizing and producing the event. For simplicity, it was decided that participants would only be those who commented on the post and actively followed the pages involved in the Giveaway at the time of the draw. The winners selection from the comment list would be automated, while the verification of page follows could easily be done manually during the draw, as there were only three eligible participants for the prizes.

### Web Scraping

Web scraping is the process of automating data extraction from web pages. Instead of manually collecting information, we use tools and scripts to efficiently perform this task, specifically using Python scripts. To execute web scraping, `HTTP` requests are sent to the web page, the `HTML` code is analyzed to identify elements of interest, and the desired data is extracted.

Not long ago, I came across some web scraping tools, including `BeautifulSoup`, which allows extracting the content of an `HTML` document from a web page and accepts attributes (`class`, `id`, etc.) as input arguments to filter the output of its search algorithm.

### Applying Web Scraping with BeautifulSoup

Let's begin exploring this tool with a rather raw example of its usage. For this example, we'll use the Instagram `Giveaway URL` to get a clearer idea of the problem we're facing. Here's a `Python` code snippet illustrating a simple web scraping focused on a web address.

```python
# PREAMBLE_______________________________
from bs4 import BeautifulSoup as Sancocho
import requests

# WEB REQUEST___________________________________________________________
Giveaway = "https://www.instagram.com/p/Cu4dTr6uBd4/?img_index=1"  # URL
wp = requests.get(Giveaway)                 # Web Request
sopa = Sancocho(wp.content, "html.parser")  # HTML Parsing

# DATA DISPLAY_____________
sopa  # Printing the result
```

After importing the necessary libraries in the code `preamble`, a request is made to the `URL` pointing to the [Instagram post](https://www.instagram.com/p/Cu4dTr6uBd4/?img_index=1). Once the result of this request is obtained in `HTML` format, the `BeautifulSoup` library is used (aliased as "Sancocho") to break down the HTML document's context into text for further processing.

It took me about 2.5 sips of tea to realize that not even Hel herself would have time to develop an algorithm or routine to filter, sort, and structure all the `HTML` jargon into usable data after seeing the result that the variable "sopa" brought with it. So, I proceeded to investigate further in order to find a similar tool that already includes some methods for structuring data to some extent during the parsing.

### Take 2! Applying Web Scraping with ~~BeautifulSoup~~ instaloader

That's when I stumbled upon this gem of a library called [`instaloader`](https://github.com/instaloader/instaloader) and this beautiful [work](https://plainenglish.io/blog/scrape-everythings-from-instagram-using-python-39b5a8baf2e5) in some corner of the internet. The work involves a routine that contains a class defined as `GetInstagramProfile`, which, in turn, has a series of methods that focus on extracting sub-elements of an Instagram profile, such as profile pictures, post hashtags, followers, post comments, and more.

After playing around with this discovery for a while, I made some changes based on my findings. Since my goal was to extract all the information from a post (in any format possible), my attention was focused on the `get_post_info_csv` method. However, I quickly noticed some issues with the definition of this method because the output data only contained a few elements from some of the posts on my Instagram profile, and the comments were nowhere to be found. After making some changes, everything worked perfectly, and we were able to extract comment data from the Instagram Giveaway post several times.

```python
# PREAMBLE______________________________________________
import instaloader                         #Instaloader
from datetime import datetime              #Calendario
from itertools import dropwhile, takewhile #Iteradores
import csv                                 #CSV

# CONSTRUCTOR_________________________________________________________________________________________
class GetInstagramProfile():
    def __init__(self) -> None:
        self.L = instaloader.Instaloader()

    def download_users_profile_picture(self,username):
        self.L.download_profile(username, profile_pic_only=True)

    def download_users_posts_with_periods(self,username):
        posts = instaloader.Profile.from_username(self.L.context, username).get_posts()
        SINCE = datetime(2021, 8, 28)
        UNTIL = datetime(2021, 9, 30)

        for post in takewhile(lambda p: p.date > SINCE, dropwhile(lambda p: p.date > UNTIL, posts)):
            self.L.download_post(post, username)

    def download_hashtag_posts(self, hashtag):
        for post in instaloader.Hashtag.from_name(self.L.context, hashtag).get_posts():
            self.L.download_post(post, target='#'+hashtag)

    def get_users_followers(self,user_name):
        '''Note: login required to get a profile's followers.'''
        self.L.login(input("input your username: "), input("input your password: ") ) 
        profile = instaloader.Profile.from_username(self.L.context, user_name)
        file = open("follower_names.txt","a+")
        for followee in profile.get_followers():
            username = followee.username
            file.write(username + "\n")
            print(username)

    def get_users_followings(self,user_name):
        '''Note: login required to get a profile's followings.'''
        self.L.login(input("input your username: "), input("input your password: ") ) 
        profile = instaloader.Profile.from_username(self.L.context, user_name)
        file = open("following_names.txt","a+")
        for followee in profile.get_followees():
            username = followee.username
            file.write(username + "\n")
            print(username)

    def get_post_comments(self,username):
        posts = instaloader.Profile.from_username(self.L.context, username).get_posts()
        for post in posts:
            for comment in post.get_comments():
                print("comment.id  : "+str(comment.id))
                print("comment.owner.username  : "+comment.owner.username)
                print("comment.text  : "+comment.text)
                print("comment.created_at_utc  : "+str(comment.created_at_utc))
                print("************************************************")

    def get_post_info_csv(self,username):
        with open(username+'.csv', 'w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            #It is required to login to get this data A94================================
            self.L.login(input("input your username: "), input("input your password: ") )
            # ===========================================================================
            posts = instaloader.Profile.from_username(self.L.context, username).get_posts()
            
            for post in posts:
                # print(idx)
                print("post date: "+str(post.date))
                print("post profile: "+post.profile)
                print("post caption: "+post.caption)
                print("post location: "+str(post.location))
                
                posturl = "https://www.instagram.com/p/"+post.shortcode
                print("post url: "+posturl)
                writer.writerow(["post",post.mediaid, post.profile, post.caption, post.date, post.location, posturl,  post.typename, post.mediacount, post.caption_hashtags, post.caption_mentions, post.tagged_users, post.likes, post.comments,  post.title,  post.url ])
            
                for comment in post.get_comments():
                    writer.writerow(["comment",comment.id, comment.owner.username,comment.text,comment.created_at_utc])
                    print("comment username: "+comment.owner.username)
                    print("comment text: "+comment.text)
                    print("comment date : "+str(comment.created_at_utc))
                print("\n\n")
                
# CALLBACKS_____________________________________________
ig = GetInstagramProfile()                 #Objeto de IG
ig.get_post_info_csv("blackphillipdesign") #Perfil de IG
```

It turns out that it is necessary to go through the login process when requesting access to a post comment section of an Instagram profile, so after these changes we could obtain what we were looking for on the first place however, there was sunshine and happiness in the meadows for only a short time.

### Gaslighting Instagram Security Measures

Fortunately, there were no issues during the execution of the Giveaway draw after this intervention. However, at the time of writing these words, it seems that everything stopped working again, along with the appearance of this new error:

```bash
JSON Query to accounts/login/: Could not find "window._sharedData" in html response. [retrying; skip with ^C]
JSON Query to accounts/login/: Could not find "window._sharedData" in html response. [retrying; skip with ^C]

QueryReturnedNotFoundException            Traceback (most recent call last)
File c:\Python310\lib\site-packages\instaloader\instaloadercontext.py:410, in InstaloaderContext.get_json(self, path, params, host, session, _attempt, response_headers)
    409 if match is None:
--> 410     raise QueryReturnedNotFoundException("Could not find \"window._sharedData\" in html response.")
    411 resp_json = json.loads(match.group(1))

QueryReturnedNotFoundException: Could not find "window._sharedData" in html response.

During handling of the above exception, another exception occurred:

QueryReturnedNotFoundException            Traceback (most recent call last)
File c:\Python310\lib\site-packages\instaloader\instaloadercontext.py:410, in InstaloaderContext.get_json(self, path, params, host, session, _attempt, response_headers)
    409 if match is None:
--> 410     raise QueryReturnedNotFoundException("Could not find \"window._sharedData\" in html response.")
    411 resp_json = json.loads(match.group(1))

QueryReturnedNotFoundException: Could not find "window._sharedData" in html response.

During handling of the above exception, another exception occurred:

QueryReturnedNotFoundException            Traceback (most recent call last)
File c:\Python310\lib\site-packages\instaloader\instaloadercontext.py:410, in InstaloaderContext.get_json(self, path, params, host, session, _attempt, response_headers)
    409 if match is None:
--> 410     raise QueryReturnedNotFoundException("Could not find \"window._sharedData\" in html response.")
...
--> 436         raise QueryReturnedNotFoundException(error_string) from err
    437     else:
    438         raise ConnectionException(error_string) from err

QueryReturnedNotFoundException: JSON Query to accounts/login/: Could not find "window._sharedData" in html response.
```

>_Sighs_...

I took a deep breath and focused on the first few lines of the message, which are printed in the console while the code is running:

```
JSON Query to accounts/login/: Could not find "window._sharedData" in html response.
JSON Query to accounts/login/: Could not find "window._sharedData" in html response.
```

Without concrete evidence but without doubt, it seemed that the `HTML` request for the login attempt wasn't successful. Instagram's application layer protocols and beyond can be seen as strict bureaucratic procedures. Essentially, there was some kind of "form" that either wasn't submitted or wasn't filled out correctly at the recipient's end during the request process. After another exhaustive 3-hour research session, I found a solution that curiously dates back to 2022, but the Giveaway was in 2023.

In essence, Instagram has security measures when requests are made using its `API`. It limits the number of requests an account can make in a given time period. Therefore, the solution involves avoiding sending login requests through a terminal and manually logging into the Instagram profile using the Firefox browser. Then, a `Python` script is executed to generate a session file copy and load it using instaloader's session instance methods, making Instagram believe it's a session recovery done in the browser — hence, `Gaslighting` brothers and sisters in Odin.

The solution consists of the following steps:

- [ ] Save the [attached code](https://raw.githubusercontent.com/instaloader/instaloader/master/docs/codesnippets/615_import_firefox_session.py) as a .py file with any name.

- [ ] Log in to the desired Instagram account using the Firefox browser.

- [ ] Once logged in, execute the code through a terminal with administrator privileges.

  ```bash
  # POWERSHELL OUTPUT 
  > python <NombreDelCodigo>.py
  
  Using cookies from <directory>
  Imported session cookie for blackphillipdesign.
  Saved session to <directory>
  ```

Here's the code in case you didn't click the URL above:

```python
from argparse import ArgumentParser
from glob import glob
from os.path import expanduser
from platform import system
from sqlite3 import OperationalError, connect

try:
    from instaloader import ConnectionException, Instaloader
except ModuleNotFoundError:
    raise SystemExit("Instaloader not found.\n  pip install [--user] instaloader")


def get_cookiefile():
    default_cookiefile = {
        "Windows": "~/AppData/Roaming/Mozilla/Firefox/Profiles/*/cookies.sqlite",
        "Darwin": "~/Library/Application Support/Firefox/Profiles/*/cookies.sqlite",
    }.get(system(), "~/.mozilla/firefox/*/cookies.sqlite")
    cookiefiles = glob(expanduser(default_cookiefile))
    if not cookiefiles:
        raise SystemExit("No Firefox cookies.sqlite file found. Use -c COOKIEFILE.")
    return cookiefiles[0]


def import_session(cookiefile, sessionfile):
    print("Using cookies from {}.".format(cookiefile))
    conn = connect(f"file:{cookiefile}?immutable=1", uri=True)
    try:
        cookie_data = conn.execute(
            "SELECT name, value FROM moz_cookies WHERE baseDomain='instagram.com'"
        )
    except OperationalError:
        cookie_data = conn.execute(
            "SELECT name, value FROM moz_cookies WHERE host LIKE '%instagram.com'"
        )
    instaloader = Instaloader(max_connection_attempts=1)
    instaloader.context._session.cookies.update(cookie_data)
    username = instaloader.test_login()
    if not username:
        raise SystemExit("Not logged in. Are you logged in successfully in Firefox?")
    print("Imported session cookie for {}.".format(username))
    instaloader.context.username = username
    instaloader.save_session_to_file(sessionfile)


if __name__ == "__main__":
    p = ArgumentParser()
    p.add_argument("-c", "--cookiefile")
    p.add_argument("-f", "--sessionfile")
    args = p.parse_args()
    try:
        import_session(args.cookiefile or get_cookiefile(), args.sessionfile)
    except (ConnectionException, OperationalError) as e:
        raise SystemExit("Cookie import failed: {}".format(e))
```

Once the session is saved in this file, you can access the desired Instagram account by using the `load_session_from_file("<IG_Username>", <"Filename">)` method. If you set the second argument to `None`, it will attempt to extract the file from the default location where the script conventionally saves it.

Broadly, it would be more generic to include an `input` to enter the Instagram account name rather than `hardcoding` it.

> Finally... _Breathes and sips tea_

After all of this, the script works as before, and you will see something similar to this printed in the console for each post found in the Instagram account:

```python
# CALLBACKS
ig = GetInstagramProfile()  # IG Object
ig.get_post_info_csv("blackphillipdesign")  # IG Profile

```

Output

```
post date: 2023-07-19 14:23:10
post profile: blackphillipdesign
post caption: • GIVEAWAY•
Para ustedes chiquitinxs en el mes de julio me uno con 2 marcas amigas y les traigo esto por muchas razones.
•Mi vuelta al Sol- Apertura de @dharmainkstudio -7 Años tatuando• 

¡Serán 3 premios para USTEDXS y las indicaciones están en las fotos!

———📍DINÁMICA 
•>Sigue las cuentas 
@sonya_xz @dharmainkstudio @blackphillipdesign @vegano.o.no 

>Etiqueta a esos 3 amigxs que siempre te acompañan en tus mejores momentos llenos de •Sonrisas•

>Comparte este post en tus historias y mencióname para poder verificar que estás participando.

_______🎁🧧
1er premio• un TATTOO de mi portafolio valorando en 300$

2do premio• Un set MERKABA portavasos para tus bebidas gracias a @blackphillipdesign realizado con impresión 3D

3er premio • Una riquísima hamburguesa ~The Barbie~ por nuestros amigos de @vegano.o.no

TIENEN HASTA EL 25 de Julio para compartir y darle like siguiendo

#giveawaypty #blackphillipdesign #veganfood
post location: None
post url: https://www.instagram.com/p/Cu4dTr6uBd4
comment username: CONFIDENTIAL
comment text: @CONFIDENTIAL @FBI @OPENUP
comment date : 2023-07-19 22:43:59
...
```

### Data Analysis

At this point, we proceed to analyze the content of the data stored in the .csv file, which will be structured as a `DataFrame`, essentially like a structured Excel table but without headers. We extract only unique usernames from the comment column to avoid repeating participants who commented more than once, and we successfully obtain the list of participants used in the raffle.

![Commments](comentarios.jpg)

### Conclusions and Future Work

While the solution demonstrated in this article proves to be viable for very specific applications, it doesn't show much promise for more ambitious and production-oriented purposes. If you wanted to containerize this scraping process to deploy a web-accessible application, you would encounter the challenge of not being able to reset the request count to Instagram's API automatically, at least not in the current state of the script. This encourages further exploration of the other methods provided by the `instaloader` library, such as the `two_factor_login()` module. This doesn't mean that during the research and development process of this project, this option wasn't explored; in fact, it was the option that seemed to make the most sense, but it didn't work correctly in any of the instances I attempted to use it.

Having the data extracted from comments and structured in a .csv file, opens up all possibilities for adding comment analysis methods and conducting more interesting dynamics that could possibly generate even more engagement from participants. For example, it'd be possible to establish a points system that generates instances in the participants pool proportional to the number of people they managed to tag in the comments (possibly limiting it to a maximum number of tags to discourage the use of spam bots) or also performing a trivia that will lead to analyzing the comments possibly using natural language processing (`NLP`) to keep it as automated as possible allowing the producers of the activity to have more freedom regarding their creativity towards this type of event.  

The truth is that after implementing this code, there is a need for further development and data analysis. While the information stored in the .csv file contains the participants and comments, we still could extract more metrics from the participant's posts and the comments themselves.

If you have any questions, suggestions, or other ideas for collaboration, please don't hesitate to contact me. I would love to hear from you and explore potential ways furthermore ideas could be taken into action.

Remember, this is just the beginning of what's possible with web scraping and data analysis. There are countless opportunities to use these techniques to gain insights, automate processes, and create innovative solutions.

Until next time,

- $A^{94}$