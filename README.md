# Introduction

[![Build Status](https://travis-ci.org/gustavosbarreto/electric-webview.svg?branch=master)](https://travis-ci.org/gustavosbarreto/electric-webview)

Electric WebView is a scriptable WebView for developers.

The WebView uses the QtWebEngine (which is based on Chromium) to render the HTML
content. There is also a simple protocol to get data, send commands and
listen for events: through Unix Socket, TCP or WebSocket. The window of WebView
does not have any UI component what you see in a standard web browser like an
address bar, status bar, navigation buttons, etc.

# Use Cases

* Web testing
  - Browser automated testing done easy.
* Page automation
  - Access and manipulate web pages using DOM API.
* Digital Signage
  - Display Web content without distractions.
* Kiosk Web Browser
  - The users can only interact with a single web application.

# Building

Before building Electric WebView you need to install GCC 6, Qt and QtWebEngine 5.6 on the system.

```sh
qmake PREFIX=/usr
make
make install
```

# Usage

### electric-webview

The WebView itself.

```
Usage: electric-webview [options]
Electric WebView is a scriptable WebView for developers.

Options:
  -h, --help                                  Displays this help.
  -v, --version                               Displays version information.
  -t, --transport <tcp|unixsocket|websocket>  Command Transport Layer to use.
  -r, --reverse <ID>                          Enable reverse mode. The ID is
                                              used to identify your session in
                                              the server.
  -s, --script <path>                         Script to run.
```

**Example:**

```sh
electric-webview -t unixsocket:/tmp/electric-webview
```

### electric-webview-ctl

A utility to interact with the WebView.

```
Usage: electric-webview-ctl [options] command
Electric WebView is a scriptable WebView for developers.

Options:
  -h, --help                                  Displays this help.
  -v, --version                               Displays version information.
  -t, --transport <tcp|unixsocket|websocket>  Command Transport Layer to use.

Arguments:
  command                                     Command to execute. Pass "-" to
                                              read from stdin.
```

**Example:**

```sh
echo "open maximized" | electric-webview-ctl -t unixsocket:/tmp/electric-webview -
```

# Transports

This section describes the available transports with their keywords, parameters, and semantics.

**`unixsocket:<filename>`**

Listens on <filename> using a UNIX domain stream socket for commands.

**`tcp:<host>:<port>`**

Listen on <host>:<port> using a TCP/IP connection for commands.

**`websocket:<host>:<port>`**

Listen on <host>:<port> using a WebSocket connection for commands.

# Commands

Electric WebView reads commands from TCP, Unix Socket or WebSocket. Each command starts
with the name of a command and is terminated by a newline. Empty line are interpreted
as end of connection.

If the command starts with `@` the command is marked as getter. See the [Getter commands][#Getter commands]
for defails.

Due to the simplicity of the protocol, it is possible to interact with the WebView
using the command-line utility GNU Netcat. However, there is `electric-webview-ctl`
utility that can be used to interact with the WebView, so you don't need extra
tools to interact with the WebView.

Simple example using the GNU Netcat utility to demonstrate how simple is the protocol:

```sh
echo "open maximized" | nc -U /tmp/electric-webview
echo "load http://github.com" | nc -U /tmp/electric-webview
```

In the below example a maximized window is open and the http://github.com is loaded.

See the [Scripting section](#Scripting) for details on how to use scripts.

## Getter commands

Commands starting with `@` are marked as getter.

Getter commands might be useful when you want to retrieve data from a Electric WebView
instance.

Example:

```sh
URL=$(echo "@current_url" | electric-webview-ctl -t unixsocket:/tmp/electric-webview -)
echo "The current URL is $URL"
```

In the below example, the response is "http://github.com".

Note that if you replace `@current_url` with `current_url` in the below example, the
response is empty because the command `current_url` is not marked as getter.

## Command response

The response of a command may vary according the type of command. If the command is
marked as getter, the response only contains the returned data from the command,
otherwise it is not a getter command, the response is composed of the command
name and returned data, separated by space.

**Note that not all commands have a response.**

## Navigation

* `load <URL>`
  - Loads the specified `URL`.

* `stop`
  - Stops loading the document.

* `reload`
  - Reloads the current document.

* `back`
  - Loads the previous document in the list of documents built by navigating links.

* `forward`
  - Loads the next document in the list of documents built by navigating links.

## Window

* `open <maximized|fullscreen>`
  - Open the window. If `maximized` is given, the window is open in maximized
    state, otherwise `fullscreen` is given, the window is open in fullscreen mode.

* `close`
  - Closes the window.

## Page

* `current_url`
  - Returns the URL of the web page currently viewed.
* `current_title`
  - Returns the title of the web page currently viewed.
* `screenshot [REGION]`
  - Take a screenshot of the web page currenly viewed and send base64 encoded JPG.
    If `[REGION]` is given, restrict the screenshot by the given region `(x,y,width,height)`.

## Content

* `set_html <string|file> <VALUE>`
  - Sets the content of the web page. If `file` is given, the `VALUE`
    is interpreted as a path to a file to load, otherwise `string` is given,
    the `VALUE` is used to set the content of the web page.

* `get_html <text|html>`
  - Retrieve the page's content formated as `html` or `text`.

* `exec_js <string|file> <VALUE>`
  - Run JavaScript code. If `file` is given, the `VALUE` is interpreted as
    a path to a file to load, otherwise `string` is given, the value `VALUE`
    is executed as JavaScript code.

* `inject_js <WORLD> <INJECTION_POINT> <string|file> <SOURCE>`
   - Injects JavaScript from `SOURCE` into `WORLD` at `INJECTION_POINT`.
     If `file` is given, the `SOURCE` is interpreted as a path to a file to load,
     otherwise `string` is given, the value `SOURCE` is executed as JavaScript code.

  Possible values for `WORLD`:
  * `main`
  * `application`
  * `user`

  Possible values for `INJECTION_POINT`:
  * `document_creation`
  * `document_ready`
  * `deferred`

  For detailed information, please refer to [Qt documentation](http://doc.qt.io/qt-5/qwebenginescript.html)

## Events

* `subscribe <VALUE>`
  - Subscribe to specified event in `VALUE`.

### Available events

* `url_changed`
  - This event is fired when the url chages.
* `title_changed`
  - This event is fired whenever the title changes.
* `load_started`
  - This event is fired when a page starts loading content.
* `load_finished`
  - This event is fired when a load of the page has finished.
* `info_message_raised <MESSAGE>`
  - This event is fired when a JavaScript program tries to print a info
    `MESSAGE` to the console.
* `warning_message_raised <MESSAGE>`
  - This event is fired when a JavaScript program tries to print a warning
    `MESSAGE` to the console.
* `error_message_raised <MESSAGE>`
  - This event is fired when a JavaScript program tries to print a error
    `MESSAGE` to the console.
* `user_activity <IDLE_TIME>`
  - This event is fired when the user interacts with the input system through
    keystrokes or mouse clicks. The `IDLE_TIME` is the milliseconds since the
    last user activity.

## Electric WebView

* `idle_time`
  - Returns the idle time from the last user activity in milliseconds.

* `block_user_activity <true|false>`
  - If `true` is given, prevents the user activity with the page until `false` is given.

* `exec_cmd <sync|async> <COMMAND> [ARGUMENTS]`
  - Run a `COMMAND` on the system with the given `ARGUMENTS`. If `async` is given,
    the command will be started detached and the output will be ignored,
    otherwise `sync` is given, the execution is blocked until the command has
    finished and the output will be send in the response message.

* `quit [code]`
  - Tells the application to exit with a return `code`.

# HTTP API

First of all, you must run the HTTP server, which is invoked with the following command:

```
$PREFIX/share/electric-webview/httpserver.py
```

> Where `$PREFIX` is the prefix where you installed Electric WebView.

See [HTTP API Endpoints](docs/http_api_endpoints.md) for a list of all available endpoints.

# Scripting

You can use scripts with Electric WebView loading the following Shell script library file:

```
$PREFIX/share/electric-webview/shellscript.sh
```

> Where `$PREFIX` is the prefix where you installed Electric WebView.

Before loading the library you must set `ELECTRIC_WEBVIEW_TRANSPORT` environment
variable to the transport layer you are using in your instance of Electric WebView.

Example:

```sh
#!/bin/sh

ELECTRIC_WEBVIEW_TRANSPORT=unixsocket:/tmp/electric-webview

. /usr/share/electric-webview/shellscript.sh

open maximized
load http://github.com
```
