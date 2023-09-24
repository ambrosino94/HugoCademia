---
title: Ey browie! ¿y si hacemos un Giveaway?
summary: Web Scraping para un Giveaway en Instagram
tags:
- Data Science
- Web Scraping
- python
date: "2023-09-24T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Set de portavasos Merkaba hecho por BlackPhillipDesign
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

En medio de una acalorada sesión de café con [Sonya](https://www.instagram.com/sonya_xz/) se juntaron las ideas, universos colisionaron y se llegó a la conclusión de que una colaboración entre [BlackPhillip](https://www.instagram.com/blackphillipdesign/) (mi negocio de diseño de producto) y [Dharma](https://www.instagram.com/dharmainkstudio/) (su estudio de tatuaje) resultaba simplemente inminente, consecutivamente como segundo punto fulminante de esta reunión espontánea, se acordó que la primera instancia de esta serie oficial de colaboraciones debería ser un `Giveaway`. 

### La problemática del Giveaway

Inmediatamente como pulga a perro callejero, me cayó la pregunta de cómo se gestionaría el conteo de participantes y en todo caso qué acciones se les iba a pedir a los participantes para luego filtrar la data bajo esos parámetros, porque es de conocimiento general que en una actividad de esta índole se acostumbra a someter a una comunidad de seguidores a realizar ciertas acciones de `engagement` tanto para considerarlos como participantes, como para aumentar el movimiento de las páginas involucradas en la organización y producción de la actividad. Por simplicidad, se decidió considerar como participantes solo aquellos que comentaron en la publicación y que activamente al momento de la tómbola estuvieran siguiendo a los perfiles involucrados en el `Giveaway`. La selección de los usuarios de la lista de comentarios se realizaría de forma automática, mientras que la verificación del seguimiento de páginas se podía fácilmente realizarse a mano al momento de la tómbola al ser solamente 3 participantes elegibles para los premios.

### Web Scraping

El `web scraping` es el proceso de automatizar la extracción de datos de  páginas web. En lugar de recopilar información manualmente, utilizamos  herramientas y scripts para realizar esta tarea eficientemente, para ser más precisos, scripts en `python`. Para llevar a cabo el `web scraping`, se envían solicitudes `HTTP` a la  página web, se analiza el código `HTML` para identificar elementos de  interés y se extraen los datos deseados.

No hace mucho, me había encontrado con algunas herramientas de `web scraping` entre esas `BeautifulSoup`, la cual básicamente permite extraer el contenido del documento `HTML`de una página web e incluso acepta atributos ( `class`, `id`, etc.) como argumentos de entrada para filtrar la salida de su algoritmo de búsqueda.

### Aplicando Web Scraping con BeautifulSoup

Empecemos explorando esta herramienta con un ejemplo bastante crudo de su uso, y pues obviamente para este ejemplo usaremos el `url` del `Giveaway` para tener una idea más clara del problema al cual nos estamos enfrentando. Justo a continuación se observa un extracto en `python` de cómo sería un `scraping` muy simple enfocado a una dirección web. 

```python
# PREAMBLE______________________________________________
from bs4 import BeautifulSoup as Sancocho #BeautifulSoup
import requests							  #Requests

# WEB REQUEST___________________________________________________________________
Giveaway = "https://www.instagram.com/p/Cu4dTr6uBd4/?img_index=1" #URL
wp = requests.get(Giveaway)										  #Solicitud Web
sopa = Sancocho(wp.content, "html.parser")						  #Desglose HTML

# DATA DISPLAY________________
sopa	#Imprimiendo resultado
```

Luego de importar las librerías necesarias en el `PREAMBLE` del código, se realiza una solicitud a la `url` que apunta a la [publicación en instagram](https://www.instagram.com/p/Cu4dTr6uBd4/?img_index=1). Una vez obtenido el resultado de esta solicitud bajo el protocolo `HTML` se usa la librería de `BeautifulSoup` bajo el alias `Sancocho` para desglosar todo el contexto del documento `HTML` en texto para luego poder procesarla de cualquier forma. 

Me tomó alrededor de 2.5 sorbos de té para llegar a la conclusión que ni el Diablo iba a tener el tiempo para desarrollar un algoritmo o rutina para filtrar, ordenar y estructurar toda la jerga `HTML` en data usable luego de ver el resultado que la variable `sopa` traía consigo, por lo que procedí a investigar más a fondo con el objetivo de encontrar una herramienta similar que ya incluya algunos métodos que al menos realicen cierto tipo de estructuración de datos cuando se haga el desglose. 

### Toma 2! Aplicando Web Scraping con ~~BeautifulSoup~~ instaloader

Fue cuando encontré en algún foro de la interwebs esta belleza de librería =>[`instaloader`](https://github.com/instaloader/instaloader)<= y esta belleza de [trabajo](https://plainenglish.io/blog/scrape-everythings-from-instagram-using-python-39b5a8baf2e5), el cual se trata de una publicación de una rutina que contiene una clase definida como `GetInstagramProfile`, que a su vez cuenta con una serie de métodos que focalizan extracciones de sub elementos de un perfil de Instagram como: Fotos de perfil, Hashtags de publicaciones, seguidores, comentarios de publicaciones, entre otros. 

Luego de jugar un rato con este hallazgo procedí a realizar algunos cambios en base a mis hallazgos. Debido a que mi objeto de interés era extraer la información completa de una publicación (en cualquier formato posible) mi atención se concentró en el método `get_post_info_csv`, pero rápidamente noté que había algunos problemas con la definición de este método ya que en la data de salida solo observaba muy pocos elementos de algunas de las publicaciones de mi perfil de Instagram y los comentarios no aparecían por ningún lado. Luego de estos siguientes cambios todo funcionó perfectamente y se logró extraer la data de los comentarios en varias ocasiones de la publicación del `Giveaway`.

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

Resulta que sí es necesario realizar el proceso de `Login` al realizar la solicitud de acceso a los comentarios de un perfil de Instagram, por lo que al realizar ese cambió se logró obtener lo que buscabamos, pero hubo sol y felicidad en las praderas solo por un tiempo.... 

### Gaslighting Medidas de Seguridad de Instagram

Por suerte no hubo ningún problema luego de esta intervención durante la ejecución de la tómbola del `Giveaway`, pero al momento de escribir estas palabras, pues aparentemente todo volvió a no funcionar junto con la nueva aparición de este error:

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

> _Suspira_... 

Procedí a respirar y enfocar mi atención en las primeras líneas del mensaje, que resultan ser las líneas que se imprimen en consola mientras se está ejecutando el código:

```
JSON Query to accounts/login/: Could not find "window._sharedData" in html response.
JSON Query to accounts/login/: Could not find "window._sharedData" in html response.
```

Sin tener evidencias concretas, pero tampoco duda alguna, parecía ser que en la solicitud `HTML` frente al intento de `Login` no hubo éxito y es que los protocolos de capa de aplicación en adelante pueden verse como estrictos trámites burocráticos y básicamente hubo algún "formulario" que, o no se entregó, o no se llenó correctamente en las puertas del destinatario durante el proceso de solicitud. Posterior a otra ardua y exhaustiva sesión de investigación de casi unas 3 horas llegué a dar con esta solución, que curiosamente data del año 2022.

En fin, en esencia Instagram cuenta con medidas de seguridad frente a las solicitudes que se hace usando su `API`, es decir que va a limitar el número de solicitudes, en un período de tiempo, que una cuenta puede realizar, por lo que la solución consta en evitar enviar solicitudes de `Login` a través de un terminal y optar por ingresar manualmente en el perfil de Instagram por medio del `browser` Firefox y aprovecharse de las cookies guardadas en la solicitud al momento de ingresar. Luego se ejecuta un script en `python` que genere un archivo copia de la sesión y poder cargarlo mediante los métodos de `instaloader` como instancia de una sesión, haciéndole creer a Instagram que se trata de una recuperación de sesión hecha en el `Browser`, ergo `Gaslighting` hermanos y hermanas de Odín.

La solución consta de los siguientes pasos

- [ ] Guardar el [código anexado](https://raw.githubusercontent.com/instaloader/instaloader/master/docs/codesnippets/615_import_firefox_session.py) en el bloque de abajo como un archivo .py bajo cualquier nombre.

- [ ]  Ingresar en la cuenta de Instagram deseada usando el buscador Firefox, sí Firefox. 

- [ ]  Ya una vez ingresado, ejecutar el código a través de un terminal con permisos de administrador.



  ```bash
  #POWERSHELL OUTPUT
  
  > python <NombreDelCodigo>.py
  
  Using cookies from <directory>
  Imported session cookie for blackphillipdesign.
  Saved session to <directory>
  ```



Acá el código por si no le espichaste al `url` arriba. 

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

Una vez guardada la sesión en este archivo es posible acceder a la cuenta de Instagram deseada a través del método `load_session_from_file("<IG_Username>", <"Filename">)`, en el cual si se coloca el segundo argumento fijado a `None`, entonces tratará de extraer el archivo desde la dirección por defecto, que convencionalmente es donde automáticamente el script de arriba guarda el archivo, por lo que la definición del método `def get_post_info_csv(self, username)`quedará siendo modificada de la siguiente manera:

```python
def get_post_info_csv(self,username):
        with open(username+'.csv', 'w', newline='', encoding='utf-8') as file:
            writer = csv.writer(file)
            #It is required to login to get this data -A94================================      
            self.L.load_session_from_file("blackphillipdesign", filename = None)
            # ================================================== =========================
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
```

De forma general, por supuesto que sería más genérico si se colocase un `input` para ingresar el nombre de la cuenta de Instagram y no colocarlo `hard coded`. 

> Por fin... _Respira y toma tésito_

Luego de todo esto, el script vuelve a funcionar como antes y vemos impreso en la consola algo similar a esto, pero para absolutamente cada post que se haya encontrado en la cuenta de Instagram con el siguiente bloque de código:

```python
# CALLBACKS_____________________________________________
ig = GetInstagramProfile()                 #Objeto de IG
ig.get_post_info_csv("blackphillipdesign") #Perfil de IG

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

### Análisis de Datos

Ya una vez en este punto, procedemos a analizar el contenido de la data almacenada en el .csv el cual estará formateada o estructurada como un `dataFrame` es decir tal cual como una tabla estructurada de Excel, pero sin encabezado. Extraemos de la columna del nombre de usuario en cada comentario solamente los usuarios únicos para evitar repetir esos participantes que comentaron más de una vez y exitosamente logramos obtener el listado de participantes que se usó en la rifa. 

![Comentarios](comentarios.jpg)



### Conclusiones y trabajos futuros

Pese a que la solución demostrada en este artículo se demuestra a sí misma como viable para aplicaciones muy puntuales, no muestra mucha promesa para propósitos más ambiciosos y orientados a producción, es decir si se desearía realizar una contenedorización de este `scraping` apuntando para desplegar una aplicación accesible en la web, se contaría con la problemática de no poder reiniciar el conteo de las solicitudes al API de Instagram de forma automática, al menos no en el estado actual del script. Esto impulsa a explorar el resto de métodos con el que cuenta la librería `instaloader` como el módulo `two_factor_login( )`, esto no quiere decir que durante el proceso de investigación y desarrollo de este proyecto no se haya intentado explorar, de hecho fue la opción que traía consigo más sentido y peso, sin embargo no llegó a funcionar correctamente en ninguna de las instancias que intenté usarla. 

Al tener la data extraída de los comentarios y estructurada en una tabla en formato .csv abre todas las posibilidades para poder agregar métodos de análisis en los comentarios y realizar dinámicas más interesantes que generen aún más `engagement` de parte de los participantes. Por ejemplo, se puede establecer un sistema a base de puntos que genere instancias del participante en el listado final proporcional a la cantidad de personas que logró etiquetar en los comentarios (posiblemente limitándolo a un máximo de etiquetas para no incitar el uso de `bots`) o realizar una competencia en los comentarios de frases, cuentos o trivias cuyas respuestas, posteriormente, puedan ser procesadas de forma automática o con cierto grado de automatización permitiéndole a los productores de la actividad un poco más de libertad en el departamento de creatividad al momento de crear estos eventos sin perder transparencia en el proceso. 

Finalmente, luego de implementar este código aún queda mucho desarrollo y análisis de dato por delante. Mientras que la data guardada en el archivo .csv contiene la información de los participantes y sus comentarios, aún podríamos extraer más metricas de cada participante y de sus propios comentarios. 

Si tienen preguntas, sugerencias u otras ideas de colaboración, no dudes en contactarme. Encantado de explorar otras potenciales formas en que se puedan llevar a cabo otras ideas. 

Hasta la próxima, 

- $A^{94}$