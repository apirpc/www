# www.apirpc.com


DSL jest do specyficznych, wyspecjalizowanych zastosowań, tutaj chciałbym mieć okazję do tworzenia w różnych formatach tych scenariuszy, by je przesyłać i wykonywać w różnych miejscach
jest coś takiego jak Apache CAMEL https://camel.apache.org/camel-k/2.0.x/

Camel is an open source integration framework that empowers you to quickly and easily integrate various systems consuming or producing data.
from('timer:tick?period=3000')
  .setBody().constant('Hello world from Camel K')
  .to('log:info')
inspirując się nim i JQUERY zrobiłem APIdsl
ale to skomplikowane, gdy musisz uczyć się tych wszystkich URI
dlatego łatwiej po prostu spiąć do kupy kod który masz lokalnie

w APIDSL chodziło o reuzycie kodu taki jaki jest
teraz mysle nad czyms co ulatwi konfiguracje detali
na początku myślałem o takim zapisie: 
+ Oczekiwany format - komenda - dana wejściowa
+ Obiekt - komenda - wartość

APIRPC:

+ format YAML, bo łatwiej to walidować
+ kilka warstw,
+ kod w zależnościach
+ uruchaminie kodu jako usługa i zarządzanie na poziomie systemu
+ SERVE możliwosć użycia z usług zdalnych
i tutaj chce to wszystko zebrać do kupy tak jak robimy to w docker compose, ale o poziom wyżej, z możliwością konfigurowania samego formatu pliku
np. na poczatku

wartswa podstawowa to warstwa systemu, dlatego każdy z tych parametrów może być obsługiwany lokalnie inaczej

to było by dobre, gdybym chciał stworzyć coś na wzór języka programowania
tutaj chodzi o tworzenie scenariuszy
np. deno to jezyk imperatywny, działający tylko z JS a mnie interesuje niezależnie od platformy
aby wykonać dowolny kod, trzeba najpier mieć do dsypozycji biblioteki
zamiast zmieniać biblioteki, chciałbym je dostosowywać w tym samym języku w którym są pisane a potem użyć tutaj w YAML bez względu na język
 język programowania typu nodej/python/... aby móc korzystać z klas, które są pośrenikami pomiędzy tym zapisem w YAML a konkretnymi bibliotekami
myślę, że lepiej rozbić te dwie kwestie, jak biblioteka i adaptacja do użycia


Ta koncepcja polega na pisaniu konwertowalnego do różnej postaci jezyka deklaratywnego, ale z zachowaniem detali,
podstawowe parametry programu to: 
+ INIT
+ IMPORT
+ SERVE
+ RUN
+ TEST
+ SET
  
postanowiłem położyć nacisk na typowanie, aby zmienne były tylko deklarowane jako publiczne
a wewnątrz używać typów danych, które są obiektami klas, dzięki czemu, można je rozwijać i tworzyć własne
IMPORT pozwala na to
staram się też nie używać rozwiązań typu pętla, czy if, aby zachować prostotę w logice
zmaiasty tego mamy strukturę z typowaniem oczekiwanych danych
apidsl bezpośrednio uzywał prostych skryptów bash-a



warstwa DSL dla komunikacji
żeby można było wymieniać dane przez HTTP poprzez komendy w konsoli/na backendzie

format DSL: apifunc.host(provider.com).data(funkcja).format(json).funkcja('POST')

format URL requestów    http:// provider.com / data_model . format

czyli przez CURL też można, np: 

```bash
GET http:// provider.com / user . json?all
DELETE  http:// provider.com / user . json ? username="enemy"
```

```bash
YAML SCAN_NETWORK 192.168.0.0/24
TXT GET /device1/api/data
JSON POST /device2/api/data { "temperature": 25 }
JSON PUT /device2/api/data { "temperature": 20 }
```


### źródło

local file
```bash
LIST READ file://domain.txt
```
or 
```bash
LIST GET file://domain.txt
```
Read remote file
plik domain.txt zawiera listę adresów URL
```bash
LIST GET https://domena.com/domain.txt
```


### file processing

pobierz adres **URL** z listy **LIST**
```bash
URL FIRST LIST
```



stwórz screenshoot z adresu **URL** elementu z listy **LIST**
```bash
IMAGE SCREENSHOT URL
```

konfiguracja
Zapisz do lokalnego pliku o nazwie hosta w formacie PNG
```yaml
IMAGE FILENAME | HOSTNAME FROM URL
IMAGE MIMETYPE PNG
```


Zapisz screenshot do folderu i nazwie pliku wczesniej zdefiniowanego

```yaml
URL FIRST LIST
IMAGE SCREENSHOT URL
FILE PATH_FOLDER file://screenshots/
FILE CONTENT IMAGE
LOG FROM FILE
```

## wnioski

wszelkie parametry podane na początku są ustalone jako default
w trakcie wykonywania można je zmienić,
np. ścieżka do pliku, może być stała
a nazwa pliku dopasowywana do ale obie te wartości i zależności można ustalić wcześniej
na koniec zapisz LOG z obiektu **FILE**

kaskadowo, kolejno
```yaml
PATH_FILE FROM file://domain.txt
LIST GET PATH_FILE
URL EACH LIST:
    IMAGE SCREENSHOT URL    
    IMAGE MIMETYPE PNG
    FILE PATH_FOLDER /screenshots/
    FILE FILE_NAME | HOST_NAME FROM URL 
    FILE CONTENT IMAGE
    LOG FROM FILE
```

```yaml
PATH_FILE: FROM file://domain.txt
LIST: GET PATH_FILE
URL: EACH LIST
LIST:
    IMAGE:
        SCREENSHOT URL    
        MIMETYPE PNG
    FILE:
        PATH_FOLDER /screenshots/
        FILE_NAME | HOST_NAME FROM URL
        CONTENT IMAGE
    LOG FROM FILE
```


```yaml
PATH_FILE: /domains/ionos.txt
LIST:
    URI PATH_FILE
    SEPARATOR NEL    
URL EACH LIST:    
    IMAGE:
        SCREENSHOT URL    
        MIMETYPE PNG
    FILE:
        PATH_FOLDER /screenshots/
        FILE_NAME | HOST_NAME FROM URL
        CONTENT IMAGE
    LOG FROM FILE
```


```yaml
PATH_FILE: /domains/ionos.txt
LIST:
    GET: PATH_FILE
    SEPARATOR: NEL    
URL EACH LIST:    
    IMAGE:
        SCREENSHOT: URL    
        MIMETYPE: PNG
    FILE:
        PATH_FOLDER: /screenshots/
        FILE_NAME:
            HOST_NAME: URL
        CONTENT: IMAGE
    LOG:        
        PATH_FOLDER: /logs/
        FILE_NAME: dsl.log
        CONTENT: FILE
```

```yaml
PATH_FILE: /domains/ionos.txt
LIST:
    GET: PATH_FILE
    SEPARATOR: NEL    
URL EACH LIST:    
    FILE:
        PATH_FOLDER: /screenshots/
        FILE_NAME:
            HOST_NAME: URL
        CONTENT:
            IMAGE:             
                SCREENSHOT: URL    
                MIMETYPE: PNG
    LOG:        
        PATH_FOLDER: /logs/
        FILE_NAME: dsl.log
        CONTENT: FILE
```




```yaml
IMPORT:
    lodash: "https://esm.sh/lodash@4.17.21"
    apirpc: "git@github.com:inframonit/bash.git"
PATH_FILE: /domains/ionos.txt
LIST:
    GET: PATH_FILE
    SEPARATOR: NEL    
URL EACH LIST:
    LOG:        
        PATH_FOLDER: /logs/
        FILE_NAME: dsl.log
        CONTENT: 
            FILE:
                PATH_FOLDER: /screenshots/
                FILE_NAME:
                    HOST_NAME: URL
                CONTENT:
                    IMAGE:             
                        SCREENSHOT: URL    
                        MIMETYPE: PNG
```

```yaml
IMPORT:
    lodash: "https://esm.sh/lodash@4.17.21"
    apirpc: "git@github.com:inframonit/bash.git"
    LIST: "git@github.com:apirpc/list.git"
    FILE: "git@github.com:apirpc/file.git"
    IMAGE: "git@github.com:apirpc/image.git"
    SCREENSHOT: "git@github.com:apirpc/screenshot.git"
SET:
    PATH_PROVIDER: "/domains/ionos.txt"
    PATH_SCREENSHOT: "/screenshots/"
    FILE_FORMAT: "png"
RUN:
    LIST:
        GET: 
            PATH_FILE: PATH_PROVIDER
        SEPARATOR: NEL    
        ITEM: URL
        EACH:    
            FILE:
                PATH_FOLDER: PATH_SCREENSHOT
                FILE_NAME:
                    HOST_NAME: URL
                CONTENT:
                    IMAGE:                                 
                        MIMETYPE: FILE_FORMAT
                        CONTENT:
                            SCREENSHOT:
                                GET: URL
                                SIZE: HD
```

```yaml
IMPORT:
    lodash: "https://esm.sh/lodash@4.17.21"
    apirpc: "git@github.com:inframonit/bash.git"
    LIST: "git@github.com:apirpc/list.git"
    FILE: "git@github.com:apirpc/file.git"
    PATH: "git@github.com:apirpc/path.git"
    IMAGE: "git@github.com:apirpc/image.git"
    SCREENSHOT: "git@github.com:apirpc/screenshot.git"
SET:
    PATH_PROVIDER: "/domains/ionos.txt"
    PATH_SCREENSHOT: "/screenshots/"
    TEST_PATH_SCREENSHOT: "/screenshots/*"
    FILE_FORMAT: "png"

TEST:
    PATH:
        EXIST: PATH_PROVIDER
        EXIST: PATH_SCREENSHOT
    FILE:
        GET: PATH_PROVIDER
        RIGHTS: READABLE
RUN:
    LIST:
        - GET: 
            PATH_FILE: PATH_PROVIDER
        - SEPARATOR: NEL    
        - ITEM: URL
        - EACH:    
            FILE:
                - PATH_FOLDER: PATH_SCREENSHOT
                - FILE_NAME:
                    HOST_NAME: URL
                - CONTENT:
                    IMAGE:                                 
                        - MIMETYPE: FILE_FORMAT
                        - CONTENT:
                            SCREENSHOT:
                                - GET: URL
                                - SIZE: HD
TEST:
    PATH:
        EXIST: TEST_PATH_SCREENSHOT            
```


## Start
załadowanie yaml
+ na podstawie IMPORT generowanie struktury projektów w .apirpc/
    + pobieranie klas i na ich podstawie walidowanie i debugowanie
+ na podstawie SET zapisanie zmiennych
+ na podstawie RUN uruchomienie wywołań
+ TEST - sprawdzenie przed i po wykonaniu



```python
class Screenshot {
    public get;
    public size;
    
    run (){
    
    }
}
```

```python
class File {
    public path_folder;
    public file_name;
    public content;

    contentFromImage() {
    }

    run (){
    
    }
}
```


```python
class Image {
    public mimetype;
    public content;
    
    run (){
    
    }
}
```


zapisywanie logów
```yaml
FILE:
    PATH_FOLDER /logs/
    FILE_NAME dsl.log
    CONTENT FILE
```

##  stara wersja:
```
ITEM FIRST LIST
IMAGE GET ITEM
FILE CREATE IMAGE
```

kaskadowo, kolejno
```bash
LIST GET /path/to/folder/images/*
FILENAME EACH LIST | FILENAME GET ITEM | TXT OCR IMAGE | FILE CREATE file://domain.txt
```


bot na strone www, desktop
[strato/login_screenshot.csv at main · botreck/strato](https://github.com/botreck/strato/edit/main/login_screenshot.csv)
```bash
BROWSER GET https://strato.pl/auth/login.html
BROWSER FOCUS | CLASS input.text.login
BROWSER WRITE | TXT GET file://strato.pl/.user
BROWSER FOCUS | CLASS input.text.password
BROWSER WRITE | TXT GET file://strato.pl/.pass
BROWSER WAIT 3000
BROWSER CLICK | XPATH input.mid.button-green-large
BROWSER CREATE file://screen.png
```


Wykonanie kodu poprzez skrypt z domyślnymi danymi:
```bash
./apirpc browser.yaml
```

lub z parametrami
```bash
./apirpc browser.yaml URL=https://strato.pl/auth/login.html PATH_OUT="/screenshots/" PATH_IN="/provider/" FOLDER_PROVIDER="strato.pl"
```


browser.yaml
```yaml
IMPORT:
    XPATH: "git@github.com:apirpc/list.git"
    TXT_FROM_FILE_PATH: "git@github.com:apirpc/file.git"
    BROWSER: "git@github.com:apirpc/path.git"

USE:
    XPATH: "class/xpath.py"
    TXT_FROM_FILE_PATH: "class/txt_from_file_path.py"
    BROWSER: "class/browser.py"    

ENV:
    XPATH: "docker image"
    TXT_FROM_FILE_PATH: "docker image"
    BROWSER: "docker image"
    
API:
    FTP:
        URI:"ftp://host:21"
        AUTH:
            PASS: .
            USER: .
        
SET:
    URL: https://strato.pl/auth/login.html
    PATH_OUT: "/screenshots/"    
    PATH_IN: "/provider/"
    FOLDER_PROVIDER: "strato.pl"
    PATH_USER:
        - "file://"
        - FOLDER_PROVIDER
        - "/.user"
    PATH_PASS:
        - "file://"
        - FOLDER_PROVIDER
        - "/.pass"

RUN:
    BROWSER:
        GET: URL
        FOCUS:
            XPATH: "input.text.login"
        WRITE:
            TXT_FROM_FILE_PATH: PATH_USER
        FOCUS:
            XPATH: "input.text.password"
        WRITE:
            TXT_FROM_FILE_PATH: PATH_PASS
        WAIT: 3000
        CLICK:
            XPATH: "input.mid.button-green-large"
        SCREENSHOT:
            - MIMETYPE: FILE_FORMAT
            - GET: URL
            - SIZE: HD
            - PATH:
                - FOLDER: PATH_SCREENSHOT
                - FILE_NAME:
                    HOST_NAME: URL
```



---



browser.yaml
```yaml
# INSTALL: wget https://www.apirpc.com/apirpc.py | python
INIT:
    RUN: "/apirpc/router.sh"
    SET: "/apirpc/config.sh"
    IMPORT: "/apirpc/import.sh"
    SERVE: "/apirpc/services.sh"
    
IMPORT:    
    XPATH:
        SOURCE: "git@github.com:apirpc/list.git"        
        ADAPTER: "python/xpath.py"
        DOCKER: "python/DOCKERFILE"
    TXT_FROM_FILE_PATH:
        GIT: "git@github.com:apirpc/txt_from_file_path.git"
        ADAPTER: "nodejs/txt_from_file_path.nodejs"
        DOCKER: "nodejs/DOCKERFILE"
    BROWSER:
        GIT: "git@github.com:apirpc/browser.git"
        ADAPTER: "java/browser.java"
        DOCKER: "java/DOCKERFILE"
    
        
SERVE:
    FTP:
        URI:"ftp://host:21"
        AUTH:
            PASS: .
            USER: .
        
SET:
    URL: https://strato.pl/auth/login.html
    PATH_OUT: "/screenshots/"    
    PATH_IN: "/provider/"
    FOLDER_PROVIDER: "strato.pl"
    PATH_USER:
        - "file://"
        - FOLDER_PROVIDER
        - "/.user"
    PATH_PASS:
        - "file://"
        - FOLDER_PROVIDER
        - "/.pass"

RUN:
    BROWSER:
        GET: URL
        FOCUS:
            XPATH: "input.text.login"
        WRITE:
            TXT_FROM_FILE_PATH: PATH_USER
        FOCUS:
            XPATH: "input.text.password"
        WRITE:
            TXT_FROM_FILE_PATH: PATH_PASS
        WAIT: 3000
        CLICK:
            XPATH: "input.mid.button-green-large"
        SCREENSHOT:
            - MIMETYPE: FILE_FORMAT
            - GET: URL
            - SIZE: HD
            - PATH:
                - FOLDER: PATH_SCREENSHOT
                - FILE_NAME:
                    HOST_NAME: URL
```








browser.yaml
```yaml
# INSTALL: wget https://www.apirpc.com/apirpc.py | python
INIT: "git@github.com:apirpc/apirpc.git"
    
IMPORT:    
    XPATH:
        SOURCE: "git@github.com:apirpc/list.git"        
        ADAPTER: "python/xpath.py"
        DOCKER: "python/DOCKERFILE"
    TXT_FROM_FILE_PATH:
        GIT: "git@github.com:apirpc/txt_from_file_path.git"
        ADAPTER: "nodejs/txt_from_file_path.nodejs"
        DOCKER: "nodejs/DOCKERFILE"
    BROWSER:
        GIT: "git@github.com:apirpc/browser.git"
        ADAPTER: "java/browser.java"
        DOCKER: "java/DOCKERFILE"
    SCREENSHOT:
        GIT: "git@github.com:apirpc/puppetter.git"
        ADAPTER: "js/browser.js"
        DOCKER: "js/DOCKERFILE"
            
SET:
    URL: https://strato.pl/auth/login.html
    PATH_OUT: "/screenshots/"    
    PATH_IN: "/provider/"
    FOLDER_PROVIDER: "strato.pl"
    PATH_USER:
        - "file://"
        - FOLDER_PROVIDER
        - "/.user"
    PATH_PASS:
        - "file://"
        - FOLDER_PROVIDER
        - "/.pass"

RUN:
    BROWSER:
        GET: URL
        FOCUS:
            XPATH: "input.text.login"
        WRITE:
            TXT_FROM_FILE_PATH: PATH_USER
        FOCUS:
            XPATH: "input.text.password"
        WRITE:
            TXT_FROM_FILE_PATH: PATH_PASS
        WAIT: 3000
        CLICK:
            XPATH: "input.mid.button-green-large"
        SCREENSHOT:
            - MIMETYPE: FILE_FORMAT
            - GET: URL
            - SIZE: HD
            - PATH:
                - FOLDER: PATH_SCREENSHOT
                - FILE_NAME:
                    HOST_NAME: URL
```






environment_config.yaml
```yaml
# INSTALL: wget https://www.apirpc.com/apirpc.py | python
INIT: "git@github.com:apirpc/apirpc.git"
    
IMPORT:    
    XPATH:
        SOURCE: "git@github.com:apirpc/list.git"        
        ADAPTER: "python/xpath.py"
        DOCKER: "python/DOCKERFILE"
    TXT_FROM_FILE_PATH:
        GIT: "git@github.com:apirpc/txt_from_file_path.git"
        ADAPTER: "nodejs/txt_from_file_path.nodejs"
        DOCKER: "nodejs/DOCKERFILE"
    BROWSER:
        GIT: "git@github.com:apirpc/browser.git"
        ADAPTER: "java/browser.java"
        DOCKER: "java/DOCKERFILE"
    SCREENSHOT:
        GIT: "git@github.com:apirpc/puppetter.git"
        ADAPTER: "js/browser.js"
        DOCKER: "js/DOCKERFILE"
            
SET:
    URL: https://strato.pl/auth/login.html
    PATH_OUT: "/screenshots/"    
    PATH_IN: "/provider/"
    FOLDER_PROVIDER: "strato.pl"
    PATH_USER:
        - "file://"
        - FOLDER_PROVIDER
        - "/.user"
    PATH_PASS:
        - "file://"
        - FOLDER_PROVIDER
        - "/.pass"

```


browser_script_run.yaml

```yaml
BROWSER:
    GET: URL
    FOCUS:
        XPATH: "input.text.login"
    WRITE:
        TXT_FROM_FILE_PATH: PATH_USER
    FOCUS:
        XPATH: "input.text.password"
    WRITE:
        TXT_FROM_FILE_PATH: PATH_PASS
    WAIT: 3000
    CLICK:
        XPATH: "input.mid.button-green-large"
    SCREENSHOT:
        - MIMETYPE: FILE_FORMAT
        - GET: URL
        - SIZE: HD
        - PATH:
            - FOLDER: PATH_SCREENSHOT
            - FILE_NAME:
                HOST_NAME: URL
```


script.apirpc
```bash
BROWSER GET URL
XPATH SET "input.text.login"
BROWSER FOCUS XPATH
BROWSER WRITE | CONTENT GET FILE | FILE READ PATH_USER
XPATH SET "input.text.password"
BROWSER FOCUS XPATH
BROWSER WRITE | TXT_FROM_FILE_PATH: PATH_PASS
BROWSER WAIT: 3000
BROWSER CLICK | XPATH: "input.mid.button-green-large"
BROWSER SCREENSHOT:
        - MIMETYPE: FILE_FORMAT
        - GET: URL
        - SIZE: HD
        - PATH:
            - FOLDER: PATH_SCREENSHOT
            - FILE_NAME:
                HOST_NAME: URL

```



start script
```bash
./apirpc script.apirpc
```
                    
### format danych
dana wyjściowa - komenda - źródło


### FILESYSTEM=HTTP alias Commands:

+ CREATE=POST
+ UPDATE=PUT

### INTERNAL Commands:
+ OCR - image processing
+ 

###  Data types:

+ LIST - internal
+ IMAGE - screenshoot
+ YAML
+ JSON
+ TXT

jako format do operacji na sieci CDN

aby łatwo dodawać i odcinać zasoby
a także do użycia lokalnie do zasobów, które mają być dostępne dla backendu, np tworzę coś na wzór elastic search dla swoich usług, domen, projektów
potrzebuję mieć przegląd biznesu z róznych API, aby móc też na tym budować logikę i strategię z planowaniem decyzji automatycznie
dlatego pomyślałem, że tak mógłbym określać zbiory zasobów
dodałbym do tego routing lokalnie w pliku i przetwarzał lokalnie w oparciu o własne zasoby lub te zdalne


```bash
GET apiRPC :// godaddy.com / domain . json?all
```

powyżej wywołanie było by przekierowywane do wewnątrz, np do pliku: local/godaddy.com/api.py a dane były by w local/godaddy.com/domain.json
chodzi o dostęp do zasobów, które można budować w formie lokalnych wywołań niezależnie od języka programowania i danych
aby można było łatwo zbudowac strukturę danych i wywoływać je na frontendzie

w formie np.  
```
http://localhost/godaddy.com/domain.json
```

czyli 
```
PROTOCOL / HOST / PROVIDER / DATA MODEL / DATA TYPE
```






+ [Examples — jsonrpcclient 4.0.0-dev documentation](https://www.jsonrpcclient.com/en/latest/examples.html)
+ [explodinglabs/jsonrpcclient: Generate JSON-RPC requests and parse responses in Python](https://github.com/explodinglabs/jsonrpcclient)

[JSON-RPC - Wikipedia](https://en.wikipedia.org/wiki/JSON-RPC)

> **JSON-RPC** is a [remote procedure call](https://en.wikipedia.org/wiki/Remote_procedure_call "Remote procedure call") [protocol](https://en.wikipedia.org/wiki/Protocol_(computing) "Protocol (computing)") encoded in [JSON](https://en.wikipedia.org/wiki/JSON "JSON"). It is similar to the [XML-RPC](https://en.wikipedia.org/wiki/XML-RPC "XML-RPC") protocol, defining only a few data types and commands. JSON-RPC allows for notifications (data sent to the server that does not require a response) and for multiple calls to be sent to the server which may be answered asynchronously.

## [Remote procedure call - Wikipedia](https://en.wikipedia.org/wiki/Remote_procedure_call)

In [distributed computing](https://en.wikipedia.org/wiki/Distributed_computing "Distributed computing"), a **remote procedure call** (**RPC**) is when a computer program causes a procedure (subroutine) to execute in a different [address space](https://en.wikipedia.org/wiki/Address_space "Address space") (commonly on another computer on a shared network), which is written as if it were a normal (local) procedure call, without the programmer explicitly writing the details for the remote interaction. That is, the programmer writes essentially the same code whether the subroutine is local to the executing program, or remote. This is a form of client–server interaction (caller is client, executor is server), typically implemented via a request–response message-passing system. In the object-oriented programming paradigm, RPCs are represented by remote method invocation (RMI). The RPC model implies a level of location transparency, namely that calling procedures are largely the same whether they are local or remote, but usually, they are not identical, so local calls can be distinguished from remote calls. Remote calls are usually orders of magnitude slower and less reliable than local calls, so distinguishing them is important.

RPCs are a form of inter-process communication (IPC), in that different processes have different address spaces: if on the same host machine, they have distinct virtual address spaces, even though the physical address space is the same; while if they are on different hosts, the physical address space is different. Many different (often incompatible) technologies have been used to implement the concept.

## [What is the difference between grpc-gateway vs Twirp RPC - Stack Overflow](https://stackoverflow.com/questions/62494187/what-is-the-difference-between-grpc-gateway-vs-twirp-rpc)

Twirp and gRPC gateway are similar. They both build API services out of a protobuf file definition.

Main differences:

- gRPC only uses protobuf over HTTP2, which means browsers can't easily talk directly to gRPC-based services.
- Twirp works over Protobuf and JSON, over HTTP 1.1 and HTTP2, so any client can easily communicate.
- gRPC is a full framework with many features. Very powerful stuff.
- Twirp is tiny and small. Only has a few basic features but it is a lot easier to manage.

reasons for gRPC over Twirp are:

+ gRPC supports streaming.
+ gRPC makes wire compatibility promises.
+ More functionality on the networking level.



## [gRPC + JSON | gRPC](https://grpc.io/blog/grpc-with-json/)

# gRPC + JSON

By [**Carl Mastrangelo**](https://carlmastrangelo.com) (Google) | Wednesday, August 15, 2018

[Contents](https://grpc.io/blog/grpc-with-json/#td-content__toc)

- [What is a Service Anyways?](https://grpc.io/blog/grpc-with-json/#what-is-a-service-anyways)
- [Sending RPCs](https://grpc.io/blog/grpc-with-json/#sending-rpcs)
- [Receiving RPCs](https://grpc.io/blog/grpc-with-json/#receiving-rpcs)
- [Optimizing the Code](https://grpc.io/blog/grpc-with-json/#optimizing-the-code)
- [Conclusion](https://grpc.io/blog/grpc-with-json/#conclusion)

So you’ve bought into this whole RPC thing and want to try it out, but aren’t quite sure about Protocol Buffers. Your existing code encodes your own objects, or perhaps you have code that needs a particular encoding. What to do?

Fortunately, gRPC is encoding agnostic! You can still get a lot of the benefits of gRPC without using Protobuf. In this post we’ll go through how to make gRPC work with other encodings and types. Let’s try using JSON.

gRPC is actually a collection of technologies that have high cohesion, rather than a singular, monolithic framework. This means its possible to swap out parts of gRPC and still take advantage of gRPC’s benefits. [Gson](https://github.com/google/gson) is a popular library for Java for doing JSON encoding. Let’s remove all the protobuf related things and replace them with Gson:

- Protobuf wire encoding
- Protobuf generated message types
- gRPC generated stub types
+ JSON wire encoding
+ Gson message types
    

Previously, Protobuf and gRPC were generating code for us, but we would like to use our own types. Additionally, we are going to be using our own encoding too. Gson allows us to bring our own types in our code, but provides a way of serializing those types into bytes.


## [Quick start | Python | gRPC](https://grpc.io/docs/languages/python/quickstart/)


This guide gets you started with gRPC in Python with a simple working example.

[Contents](https://grpc.io/docs/languages/python/quickstart/#td-content__toc)

- - [Prerequisites](https://grpc.io/docs/languages/python/quickstart/#prerequisites)
        - [gRPC](https://grpc.io/docs/languages/python/quickstart/#grpc)
        - [gRPC tools](https://grpc.io/docs/languages/python/quickstart/#grpc-tools)
    - [Download the example](https://grpc.io/docs/languages/python/quickstart/#download-the-example)
    - [Run a gRPC application](https://grpc.io/docs/languages/python/quickstart/#run-a-grpc-application)
    - [Update the gRPC service](https://grpc.io/docs/languages/python/quickstart/#update-the-grpc-service)
    - [Generate gRPC code](https://grpc.io/docs/languages/python/quickstart/#generate-grpc-code)
    - [Update and run the application](https://grpc.io/docs/languages/python/quickstart/#update-and-run-the-application)
        - [Update the server](https://grpc.io/docs/languages/python/quickstart/#update-the-server)
        - [Update the client](https://grpc.io/docs/languages/python/quickstart/#update-the-client)
        - [Run!](https://grpc.io/docs/languages/python/quickstart/#run)
    - [What’s next](https://grpc.io/docs/languages/python/quickstart/#whats-next)

# Quick start

This guide gets you started with gRPC in Python with a simple working example.

### Prerequisites[](https://grpc.io/docs/languages/python/quickstart/#prerequisites)

- Python 3.7 or higher
- `pip` version 9.0.1 or higher

If necessary, upgrade your version of `pip`:

    $ python -m pip install --upgrade pip
    

If you cannot upgrade `pip` due to a system-owned installation, you can run the example in a virtualenv:

    $ python -m pip install virtualenv
    $ virtualenv venv
    $ source venv/bin/activate
    $ python -m pip install --upgrade pip
    

#### gRPC[](https://grpc.io/docs/languages/python/quickstart/#grpc)
> 
> Install gRPC:
> 
>     $ python -m pip install grpcio
>     
> 
> Or, to install it system wide:
> 
>     $ sudo python -m pip install grpcio
>     
> 
> #### gRPC tools[](https://grpc.io/docs/languages/python/quickstart/#grpc-tools)
> 
> Python’s gRPC tools include the protocol buffer compiler `protoc` and the special plugin for generating server and client code from `.proto` service definitions. For the first part of our quick-start example, we’ve already generated the server and client stubs from [helloworld.proto](https://github.com/grpc/grpc/tree/v1.54.0/examples/protos/helloworld.proto), but you’ll need the tools for the rest of our quick start, as well as later tutorials and your own projects.
> 
> To install gRPC tools, run:
> 
>     $ python -m pip install grpcio-tools





## [Web API Architecture Style](https://restlet.talend.com/web-api-style/)

There is no reason to oppose both styles as they are not solving the same problems, even though they are both deeply connected with the Web. The figure below represents their main differences.

![][img1]

Both styles are complementary and can be used together, for example when building a single page web application where the main HTML is loaded through a regular web page (REST style), including some JavaScript (code on demand constraint). This code is then interpreted by the web browser to make AJAX calls back to a Web API, exchanging predefined JSON representations.

Let’s also mention hyperdata as a second form of hypermedia with hypertext, that also offers a comprehensive application of REST. This is the world of the Semantic Web and especially the pragmatic Linked Data movement based on RDF and related media types such as Turtle, JSON-LD or HAL.

In other situations, where the client isn’t a web browser but a native mobile app, a connected device or a program written by a partner to integrate your web site with their own, you only rely on the Web API style, which is fine again. Let’s now step back and see where these new forms of web architectures will lead us.

## 7\. Comparison with RPC

![][img2]



[img1]: data:image/png;base64,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
[img2]: data:image/png;base64,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
