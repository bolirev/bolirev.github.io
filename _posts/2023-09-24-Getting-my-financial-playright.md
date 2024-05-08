
# Getting my financial reports with playwright, automatically

Getting my financial reports with playwright, automatically

As an hobbyist in stock market investments with a fervor for data engineering and analysis, I found myself driven to monitor and optimize my financial strategies. To achieve this, I required access to underlying data such as stock quantities, prices, and other relevant information at different time intervals. Initially, I adhered to a Sunday morning routine of manually navigating websites to download the pertinent data. However, this method proved restrictive in terms of analysis and became a hindrance to the enjoyment of my Sunday mornings. Frustrated by these limitations, I decided to automate the entire process.

Regrettably, my broker did not provide an API for my needs (or did not find it yet). Fortunately, Playwright attracted my attention as the solution to my hindrance. In this article, I aim to share my journey in developing an automated data downloading pipeline. Financial websites often implement two-factor authentication (2FA), initially appearing as a significant hurdle. However, as we will see, overcoming this challenge only required a one-time setup, offering a streamlined approach for downloading my reports.

**What is playwright**

Before we embark on the journey, most of this was possible thanks to a tool named playwright. Playwright is a Node.js library for automating web browsers, including Chromium (Google Chrome), Firefox, and WebKit (Safari). It provides a high-level API for browser automation, making it easier for developers to perform tasks like web scraping, automated testing, and interaction with web pages. Playwright is known for its cross-browser support, robustness, and support for headless and headful browser modes. It also allows developers to script interactions with web pages, capture screenshots and videos, and perform a wide range of browser automation tasks. It’s a powerful tool for building web automation scripts and testing web applications across multiple browsers.

***Choice of API and language***

At the time of writting, playwright offers API for:

* Node.js

* Python

* Java

* .Net

In this tutorial we will use the Python-API

***Choosing between Sync vs Async API:***

Playwright offers both synchronous (sync) and asynchronous (async) APIs for browser automation.

The sync API may be more intuitive for developers who are used to synchronous programming, but it may block the execution of the script during each operation.

The async API allows for non-blocking execution, making it more suitable for tasks that require concurrency, responsiveness, and handling multiple asynchronous actions simultaneously.

In this tutorial we will use the sync API because the task neither need to be responsive nor concurrent.

### **Login and two factor authentification**

**Login page or login pages**

    from playwright.sync_api import sync_playwright
    
    with sync_playwright() as playwright:
        browser = playwright.firefox # or "firefox" or "webkit".
        browser = browser.launch(headless = False, slow_mo = 50)
        context = browser.new_context()
        page = context.new_page()
        page.goto("https://webtrading.onvista-bank.de/login")
        page.pause()

![](https://cdn-images-1.medium.com/max/3786/1*-6Y_7o25p0YR5qYv2FTj2g.png)

On the left side we have the browser started by playwright and the page it reached. We can see the results because we set headless to False, and ended the example above with a page.pause().

On the right side we have an handy tool provided by playwright. We can click for example on the record button, and navigate through the page. By doing so we obtain the list of command that are potentially working.

There is however a catch on this webpage. From time to time we have a security keyboard blocking the access to the block for login to the page.

![](https://cdn-images-1.medium.com/max/3864/1*A1b4de4QYtNkn-uwrmxhEQ.png)

We can however attempt to click on the linked once it is displayed.

    # (...)
    with sync_playwright() as playwright:
        # (...) page.goto("https://webtrading.onvista-bank.de/login")
        page.wait_for_load_state('networkidle') # To make sure that link is clickable
        try: 
            page.get_by_role("link", name="Sicherheitstastatur ausblenden").click()
        except Exception as e:
            print(e)
        page.pause()

**Login with username and password**

![Finding the element for login and password using the webpage inspector](https://cdn-images-1.medium.com/max/3372/1*Yg60BQ5a6_UMuHzlHzy-Rw.png)*Finding the element for login and password using the webpage inspector*

For login to the webpage, the python script needs to be aware of our login and password. These should be securely stored. For the example below, we will simply store them in a json file named in ‘secrets.config’.

    {"secret": 
        {"login": "MyLogin", 
         "password": "MyPassword"}
    }

This file is then loaded at the beginning of the script.

By using

    import json
    from playwright.sync_api import sync_playwright
    
    with open('secrets.config', 'r') as f:
        data = json.load(f)
        secret = data['secret']
    
    with sync_playwright() as playwright:
        browser = playwright.firefox # or "firefox" or "webkit".
        browser = browser.launch(headless = False, slow_mo = 50)
        context = browser.new_context()
        page = context.new_page()
        page.goto("https://webtrading.onvista-bank.de/login")
        page.wait_for_load_state('networkidle') # To make sure that link is clickable
        try: 
            page.get_by_role("link", name="Sicherheitstastatur ausblenden").click()
        except Exception as e:
            print(e)
        page.fill('#login',secret['login'])
        page.fill('#password',secret['password'])
        page.click('#performLoginButton')
        page.pause()

Well … the webpage is asking for a 2FA identification. This is reassuring and expected from a banking page, but it makes the process at first seem impossible to go around.

**2FA with cookies**

At that point, I was a bit concerned. But I knew that I don’t need to require a security code every time I want to access the page. It means that the 2FA is stored, probably in a cookie 🤔.

Indeed comparing the cookies stored before and after the 2FA, we see new ones arrived :)

![Before the 2FA](https://cdn-images-1.medium.com/max/4124/1*6aE_Aa4VAFVIW6a1p8Xd1w.png)*Before the 2FA*

![After the 2FA](https://cdn-images-1.medium.com/max/4124/1*U--fmZTdik3LzcPRNSIM2Q.png)*After the 2FA*

One of the two will expire in the future, but in the far-far future (one year) in comparison to every Sundays. Here I did mark the expiration date in my calendar to renew the cookie before a future crash.

*Note: name and value containing sensitive information has been changed*

Playwright can store and load cookies, so let’s adapt the script to handle cookies, i.e.:

1. Goes through the 2FA procedure and store the cookies when they can’t be loaded

1. Add the cookies to the page, otherwise.

    import os
    import pickle
    #(...)
    cookie_file = 'secrets.pickle'
    
    with sync_playwright() as playwright:
        #(...) context = browser.new_context()
        if os.path.exists(cookie_file):
            with open(cookie_file, 'rb') as handle:
                mycookies = pickle.load(handle)
                context.add_cookies(mycookies)
        else:
            mycookies = None
        # page = context.new_page() (...)
        if mycookies is None:
            answ = input('Did you do the verification? (y/N)')
            if answ.lower()!='y':
                exit()
            mycookies = context.cookies()
            with open(cookie_file, 'wb') as handle:
                pickle.dump(mycookies, handle)
        page.pause()

![](https://cdn-images-1.medium.com/max/3770/1*PJxweVUr6riTiDnjcmkgCw.png)

Hurray it works :) The rest was pretty straight forward. Going to a page and clicking on the correct button.

### Conclusion

We’ve created a Python script designed to access a website that demands a username, password, and a two-factor authentication (2FA) code. The 2FA process arises only when using a standard web browser without private mode. This suggests that the 2FA is saved within the browser, likely through the use of cookies. By transferring the cookie information to Playwright, we’ve effectively gained access to the webpage without encountering the 2FA requirement, or, to be more precise, by using a pre-stored 2FA code.

The development of this script was facilitated by the ‘page.pause()’ command and by initiating the browser in a non-headless mode (i.e., with ‘headless=False’).

In addition to the ‘page.pause’ feature, we examined the page source to identify potentially relevant IDs, such as #login and #password, in order to locate the fields necessary for entering the login credentials.

To fully leverage automation, we could further exploit Playwright for accessing the specific documents we’re interested in. This aspect hasn’t been covered in this notebook because it depends on your unique requirements and is, among other factor, site-specific. However, this future development will likely follow a similar process, and you now have the knowledge to continue adapting it for your specific needs.

### Appendix

**Configuration**

I needed to install a few packages to make the full down-loader working. The configuration of my poetry is as follow.

    ```
    [tool.poetry]
    name = "playwright-downloader-onvista"
    version = "0.1.0"
    description = "My personal downloader for documents"
    authors = ["O. Bertrand <xyz@abs.com>"]
    
    [tool.poetry.dependencies]
    python = "^3.9"
    pytest-playwright = "*"
    beautifulsoup4 = "*"
    pandas = "*"
    lxml = "*"
    numpy = "*"
    
    [tool.poetry.group.tutorial]
    optional = true
    
    [tool.poetry.group.tutorial.dependencies]
    jupyter = "*"
    
    [build-system]
    requires = ["poetry-core>=1.2.0"]
    build-backend = "poetry.core.masonry.api"
    ```

After installing poetry dependencies, you also need to install the driver for playwright.

    playwright install

**Hardware**

I developed the code on Ubuntu, and let it run on a Raspberry PI.
