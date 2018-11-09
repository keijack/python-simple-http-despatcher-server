# python-simple-http-server

## Discription

This is a simple http server, use MVC like design.

## Support Python Version

Python 2.7 / 3.6+ (It should also work at 3.5, not test)

## Why choose

* Lightway.
* Filter chain support.
* Spring MVC like request mapping.
* Easy to use.
* Free style controller writing.

## How to use

### Install

```shell
pip install simple_http_server
```

### Write Controllers

```python

from simple_http_server import request_map
from simple_http_server import Response
from simple_http_server import MultipartFile
from simple_http_server import Parameter
from simple_http_server import Parameters
from simple_http_server import Header
from simple_http_server import JSONBody
from simple_http_server import HttpError
from simple_http_server import StaticFile
from simple_http_server import Headers
from simple_http_server import Cookies
from simple_http_server import Cookie


@request_map("/index")
def my_ctrl():
    return {"code": 0, "message": "success"}  # You can return a dictionary, a string or a `simple_http_server.simple_http_server.Response` object.


@request_map("/say_hello", method=["GET", "POST"])
def my_ctrl2(name, name2=Parameter("name", default="KEIJACK")):
    """name and name2 is the same"""
    return "<!DOCTYPE html><html><body>hello, %s, %s</body></html>" % (name, name2)


@request_map("/error")
def my_ctrl3():
    return Response(status_code=500)


@request_map("/exception")
def exception_ctrl():
    raise HttpError(400, "Exception")

@request_map("/upload", method="GET")
def show_upload():
    root = os.path.dirname(os.path.abspath(__file__))
    return StaticFile("%s/my_dev/my_test_index.html" % root, "text/html; charset=utf-8")

@request_map("/upload", method="POST")
def my_upload(img=MultipartFile("img")):
    root = os.path.dirname(os.path.abspath(__file__))
    img.save_to_file(root + "/my_dev/imgs/" + img.filename)
    return "<!DOCTYPE html><html><body>upload ok!</body></html>"


@request_map("/post_txt", method="POST")
def normal_form_post(txt):
    return "<!DOCTYPE html><html><body>hi, %s</body></html>" % txt

@request_map("/tuple")
def tuple_results():
    # The order here is not important, we consider the first `int` value as status code,
    # All `Headers` object will be sent to the response
    # And the first valid object whose type in (str, unicode, dict, StaticFile, bytes) will
    # be considered as the body
    return 200, Headers({"my-header": "headers"}), {"success": True}

"""
" Cookie_sc will not be written to response. It's just some kind of default
" value
"""
@request_map("tuple_cookie")
def tuple_with_cookies(all_cookies=Cookies(), cookie_sc=Cookie("sc")):
    print("=====> cookies ")
    print(all_cookies)
    print("=====> cookie sc ")
    print(cookie_sc)
    print("======<")
    import datetime
    expires = datetime.datetime(2018, 12, 31)

    cks = Cookies()
    # cks = cookies.SimpleCookie() # you could also use the build-in cookie objects
    cks["ck1"] = "keijack"
    cks["ck1"]["path"] = "/"
    cks["ck1"]["expires"] = expires.strftime(Cookies.EXPIRE_DATE_FORMAT)
    # You can ignore status code, headers, cookies even body in this tuple.
    return Header({"xx": "yyy"}), cks, "<html><body>OK</body></html>"

"""
" If you visit /a/b/xyz/x，this controller function will be called, and `path_val` will be `xyz`
"""
@request_map("/a/b/{path_val}/x")
def my_path_val_ctr(path_val=PathValue()):
    return "<html><body>%s</body></html>" % path_val
```

### Write filters

```python
from simple_http_server import filter_map

# Please note filter will map a regular expression, not a concrect url.
@filter_map("^/tuple")
def filter_tuple(ctx):
    print("---------- through filter ---------------")
    # add a header to request header
    ctx.request.headers["filter-set"] = "through filter"
    if "user_name" not in ctx.request.parameter:
        ctx.response.send_redirect("/index")
    elif "pass" not in ctx.request.parameter:
        ctx.response.send_error(400, "pass should be passed")
        # you can also raise a HttpError
        # raise HttpError(400, "pass should be passed")
    else:
        # you should always use do_chain method to go to the next
        ctx.do_chain()
```

### Start your server

```python
# If you place the controllers method in the other files, you should import them here.

import simple_http_server.server as server
import my_test_ctrl


def main(*args):
    server.start()

if __name__ == "__main__":
    main()
```

## Problems

### Unicode supporting

Although I have tried to fixed the unicode problem in python 2.7, it still may cause some problems for the reason that python 2.7 is quite unfriendly for unicodes. The best way to ensure unicode works is to use python 3.6+

### Multipul threading safety

For this is a SIMPLE http server, I have not done much work to ensure multipul threading safety. It may cause some problem if you trid to write data to `Request` and `Response` objects in multipul threads in one request scope (including in filters and controller functions).