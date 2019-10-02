WSGI 
=============
WSGI interface has two sides:
- the "server" or "gateway" side
- the "application" or "framework" side

#### function of servers
- servers or gateways will require an application's deployer to write a short
script to create an instance of the server or gateway, and supply it with the
application object

- use configuration files or other mechanisms to specify where an application
object should be imported from, or otherwise obtained


##The Application/Framework Side

**Note**WSGI is a tool for framework and server developers, and is not intended
to directly support application developers.

example with **function** and **class**:
```python
def simple_app(environ, start_response):
    """Simplest possible application object"""
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello world!\n']


class AppClass:
    """Produce the same output, but using a class

    (Note: 'AppClass' is the "application" here, so calling it
    returns an instance of 'AppClass', which is then the iterable
    return value of the "application callable" as required by
    the spec.

    If we wanted to use *instances* of 'AppClass' as application
    objects instead, we would have to implement a '__call__'
    method, which would be invoked to execute the application,
    and we would need to create an instance for use by the
    server or gateway.
    """

    def __init__(self, environ, start_response):
        self.environ = environ
        self.start = start_response

    def __iter__(self):
        status = '200 OK'
        response_headers = [('Content-type', 'text/plain')]
        self.start(status, response_headers)
        yield "Hello world!\n"
```

##The Server/Gateway Side

The server or gateway invokes the application callable once for each request
it receives from an HTTP client, that is directed at the application

**Note:** uncaught exception will be dumped to sys.stderr and logged by the
web server

```python
import os, sys


def run_with_cgi(application):
    
    environ = [os.environ.items()]
    environ['wsgi.input'] = sys.stdin
    environ['wsgi.errors'] = sys.stderr
    environ['wsgi.version'] = (1,0)
    environ['wsgi.multithread'] = False
    environ['wsgi.multiproccess'] = True
    environ['wsgi.run_once'] = True

    if environ.get('HTTPS, 'off') in ('on', '1'):
       environ['wsgi.url_scheme'] = 'https'
    else:
       environ['wsgi.url_scheme'] = 'http' 

    headers_set = []
    headers_sent = []

    def write(data):
        if not headers_set:
            raise AssertionError('write() before start_response()')

        elif not headers_sent:
            status, response_headers = headers_sent[:] = headers_set
            for header in response_headers:
                 sys.stdout.write('%s: %s\r\n' % header)
             sys.stdout.write('\r\n')
    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[0], exc_info[1], exc_info[2]
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]
        return write

    result = application(environ, start_response)
    try:
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```
##### environ
**Note:** The environ dictionary contains `CGI` environment
variables (Common Gateway Interface).
It consists of :
- REQUEST_METHOD : verbs such as "GET" or "POST"
- SCRIPT_NAME :  initial portion of the request URL's "path"
- PATH_INFO : remainder of the request URL's "path"
- QUERY_STRING : ortion of the request URL that follows the "?"
- CONTENT_TYPE : The contents of any Content-Type 
- SERVER_NAME, SERVER_PORT : combined with SCRIPT_NAME and PATH_INFO
- SERVER_PROTOCOL : like "HTTP/1.0" or "HTTP/1.1"
- HTTP_ Variables : names begin with "HTTP_"
**Note:**In addition, if SSL is in use, the server or gateway should also 
provide as many of the Apache SSL environment variables
**Note:**A WSGI-compliant server or gateway should document what variables it
provides

##### start_response()
arguments:
- **status argument** is an HTTP "status" string like "200 OK" or "404 Not Found"
- **response_headers** argument is a list of (header_name, header_value) tuples
- **exc_info** is a  list of exception trapped by the application

```python
def start_response(status, response_headers, exc_info=None):
    if exc_info:
         try:
             # do stuff w/exc_info here
         finally:
             exc_info = None    # Avoid circular ref.
```

- must not actually transmit the response headers. Instead, it must store them
for the server or gateway to transmit only after the first iteration of the
application return value that yields a non-empty string, or upon the
application's first invocation of the write() callable
## Middleware: Components that Play Both Sides

- Routing a request to different application objects based on the target URL,
after rewriting the environ accordingly.
- Allowing multiple applications or frameworks to run side-by-side in the same
prashasadiiranrocess
- Load balancing and remote processing, by forwarding requests and responses
over a network
- Perform content postprocessing, such as applying XSL stylesheets



##Details
-  A server or gateway must invoke the application object using positional (not
keyword) arguments. (E.g. by calling result = application(environ
,start_response)
- The start_response callable must return a write(body_data) callable that
takes one positional parameter: a string to be written as part of the HTTP
response body


