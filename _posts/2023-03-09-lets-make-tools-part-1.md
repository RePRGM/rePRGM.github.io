---
layout: post
title: Lets Make Tools – Simple Directory Brute Forcing
---

![Lets Make Tools](/assets/lets_make_malware_header.jpg)

# Preamble
Nim is a terrific language for malware development, but it is also a fantastic language for general purpose development. As we alluded to in our post titled [Why Nim](https://), the downside of Nim is that there are not a lot of tutorials, guides, or even code samples to learn from. 

So, let’s rectify that. As an added bonus, we will also be contributing to the (semi-) legitimate use of Nim for software development (for once). 

# Overview

In this post, we will be going over how to make a simple directory brute force tool. For this project we have a few goals. 

First, we want it to be under 100 lines of code. In the grand scheme of things 100 lines of code is barely anything and can be quick to write from start to finish. Especially with Nim as our programming language.

Although, that also means we will not have very many features. But that is okay as this project is meant to be more of a proof-of-concept others can take, improve, and learn from.

Second, we want it to be beautiful. Nim’s standard library provides us with the means to make a nice console application with things like colored and blinking text, erasing lines, and presenting progress bars. We will only be making use of the colored output for this project, but that will be more than enough to make it beautiful.

Third, we want it to be fast. Language choice _does_ matter, despite what we may have been told. And for a project like this, while we could choose any language, ideally, we want something that can make use of multithreading, multiprocessing, or asynchronous operations.

That means, ideally, no Python which does not _really_ have multithreading. 

_Don’t worry Python, we still love you._

# Getting Started

To start, we will create our `main` function like so:

`proc main(): void =`

In Nim, functions, or procedures rather, are created with the `proc` keyword and then the identifier. Inside the parenthesis, just like other languages, we can specify arguments and their types. We put a colon afterwards and declare the return type at the end. 

Nim is like “newer” languages such as Go and Rust in this regard. Types are declared at the end, instead of the front. 

We end the line with an equal sign and go to a new line where we can begin filling out the body of our procedure. Nim, much like Python, likes its whitespace, so from here on each line will begin with either spaces or tabs. 

Inside our body, we will add niceties such as displaying information to the user like the domain name they entered, the wordlist being used, number of threads, etc etc.

Finally, we will add the main logic of our tool. This involves calling a procedure to parse the provided wordlist and another call to begin brute forcing directories. It is our brute force procedure that will be threaded. 

Our complete main procedure will look like this:

```nim
proc main(): void = 
  echo "[*] Domain: ", params[0]
  echo "[*] Wordlist: ", params[1]
  echo "[*] Threads: ", params[2]
  echo "[*] Hidden: 404\n[*] Timeout: 10s"
  stdout.styledWriteLine(fgWhite, "[*] Status Codes: [", fgGreen, "200, 201, ", fgYellow, "301, 302, 307, 308, ", fgRed, "401, 403, 405, 410", fgWhite, "]")
  
  var subWordLists = parseWordList(params[1], parseInt(params[2]))
  echo fmt """
+=======================================================================================+
Starting NimBust: {getDateStr()} at {getClockStr()}
+=======================================================================================+"""
  for i in 0 ..< parseInt(params[2]): spawn bruteForce(subWordLists[i], params[0])
  sync()
  echo fmt """
+=======================================================================================+
Finished NimBust: {getDateStr()} at {getClockStr()}
+=======================================================================================+"""
```

`params` is a variable that is declared at the top (globally) like so:

`var params = commandLineParams()`

`commandLineParams` is a procedure that does exactly what it sounds like: gets the command line parameters and returns them as a sequence of strings. 

Sequences are variable-length arrays. They are zero-indexed just like arrays in other languages. Nim, of course, also has an array type that is fixed length like usual.

You will notice we did not declare a type for the `params` variable, and that is because Nim is smart enough to infer that information from the return type of our procedure call. 

`styledWriteLine` is where our pretty colors come from. This procedure comes from the terminal module, which means we need to import it.

Importing modules looks like this: `import std/terminal`.

On the next line, we make a call to a procedure called `parseWordList` that we still need to create and save its result to a variable called `subWordLists`. We name this variable `subWordLists` because our `parseWordList` procedure is going to split up the provided wordlist into evenly distributed ones we can pass to the brute force procedure later on. 

We will end up running this procedure multiple times in different threads, so by splitting up our wordlist, we give each thread an even amount of work to do.

Getting back to our `main` procedure, we use the `echo` procedure in combination with the `fmt` procedure to print out more niceties to the screen. This time we will display the start date and time (and later, the finish date and time) using the `getDateStr` and `getClockStr` procedures from the times module.

`fmt` comes from the strformat module, so we import that as well. 

Next we have a for loop that goes from 0 to the number of threads provided and calls our `bruteForce` procedure in a new thread with the `spawn` procedure. Technically, here we make use of a threadpool (which is also the module name this procedure comes from). 

The `sync` procedure (also from the threadpool module) just tells our main thread to wait for the others to finish. And that covers our `main` procedure. 

Note that a main procedure is not required. It just makes for cleaner code. 

# when isMainModule

There is one other related block of code to our `main` procedure. As we mentioned, a main procedure is not required. As such, it does not get called by anyone other than us. So, we use `when isMainModule` to implement such application logic. 

This line is essentially the same as Python’s `if __name__ == __main__` line. It says “_only run this code block when it is not being imported_”.

Not required, but it leads to cleaner code.

# parseWordList()

Next, we will look at creating our parser procedure. The completed procedure will look like this:

```nim
proc parseWordList(wordlist: string, thrNum: int): seq[seq[string]] =  
  var completeList: seq[string]
  try: 
    completeList = wordlist.lines.toSeq.filterIt(
       not it.startsWith("#") and not it.isEmptyOrWhitespace
      )
  except IOError:
    stdout.styledWriteLine(fgRed, "(Error)", fgDefault, fmt "Cannot open {wordlist}")
    quit(1)
  return completeList.distribute(thrNum)
```

As short as this code is, there is quite a bit going on here. 

First, we make a variable called completeList that will be a sequence of strings. 

Then, we set it to equal to `wordlist.lines.toSeq.filterIt(
not it.startsWith(“#”) and not it.isEmptyOrWhitespace
)`.

Let’s break that down. `wordlist` is a string parameter of the procedure. This is the user provided wordlist. 

`lines` is not a procedure, but an iterator. It takes a string parameter called filename. This is the `wordlist` parameter of our `parseWordList` procedure. Then, it opens the file of that name or errors out if it cannot (hence why we put it in a `try` block), and iterates through it line by lines. 

Note: we can use iterators in loops as well. They are more or less the same as Python’s iterators.

Next, we use the `toSeq` template (not a procedure) to turn an iterator, `lines` in this case, into a sequence. A sequence of what? Whatever the iterator returns. That is essentially what a template does. They can be thought of (in most statically typed languages that have them, not just Nim) as generic functions. Or as functions that don’t have a specific return type or have parameters of any specific type.

`filterIt` is another template. It lets us apply certain logic to each item. In this case, each item within our sequence. As you can see, we are filtering out blank lines and lines that begin with the # symbol. These are usually comments. 

Note: `it` is a special variable that comes with `filterIt` that we use with it.

Both `filterIt` and `toSeq` come from the sequtils module.

Next, we have an `except` block that will display our custom error message in pretty colors and quit gracefully.

Finally, we have our call to `distribute` (also from the sequtils module). Here, we take our new filtered wordlist (`completeList`) and split it evenly by the number of threads. If there are 100 words in our list and we want 10 threads, we will get a sequence of 10 sequences of 10 words (strings) each. We can also call it a 2-D sequence. And that is also the return type of our procedure!

# bruteForce()

Lastly, we have the meat and potatoes of this whole project: our bruteForce procedure. This one is a bit different as we have to include the `thread` pragma in the procedure signature. The full procedure will look like this:

```nim
proc bruteForce(wordlist: seq[string], baseUrl: string): void {.thread.} =
  var 
    url: string
    response: Response
  
  let client = newHttpClient(userAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:110.0) Gecko/20100101 Firefox/110.0", sslContext=newContext(verifyMode=CVerifyPeer), timeout=10000)
  
  for word in wordlist:
    try:
      url = fmt "{baseUrl}/{word}"
      response = client.head(url)
      defer: client.close()
      
      if response.status.contains(@[“200”, “201”].HttpHeaderValues): stdout.styledWriteLine(fgGreen, fmt "({response.status.splitWhitespace()[0]})", fgWhite, fmt "\t/{word}")        
      elif response.status.contains(@[“301”, “307”, “308”].HttpHeaderValues) or “302” in response.status: stdout.styledWriteLine(fgYellow, fmt "({response.status.splitWhitespace()[0]})", fgWhite, fmt "\t/{word}")
                
      elif "404" notin response.status: stdout.styledWriteLine(fgRed, fmt "({response.status.splitWhitespace()[0]})", fgWhite, fmt "\t/{word}")
        
    except: 
      var error = getCurrentException()
      stdout.styledWriteLine(fgRed, fmt "/{word}: {error.msg}")
```

For this section we will need to import httpclient. We will also need to import specific things from httpcore and net. For those we can use the following: `from std/net import newContext, SslCVerifyMode, TimeoutError` and `from std/httpcore import HttpHeaderValues, contains`

As an overview, all we are doing in this procedure is looping through every word in our wordlist, prepending our website url to it, making a request, and checking the response status code.

The `newHttpClient` procedure should be simple to understand. It is how we create an HTTP client (instance).

There are optional parameters. The important one here is the sslContext parameter. This tells the client what to do with SSL/TLS certifications. If we wanted to ignore them, we can use this parameter to do so.

Moving on, we have the `head` procedure. It has two parameters. The first is the HttpClient instance. This is from our call to `newHttpClient`.The second is the url. The `head` procedure returns a Response object. 

This Response object contains fields such as the status code. 

On the next line, the `defer` statement works much like Python’s `with` statement. It lets Nim handle, in this case, closing the connection (which is what the `close` procedure does) when we are down with it.

And finally our last few lines. `Contains` and the `in` statement are actually very similar. `in` in Nim is actually just syntax “sugar” for `contains`. Both just look for a given string. We could also use the `==` operator here, but we would have to provide the _full_ string (as in “200 OK”).

Also, _this_ `contains` is overloaded from the “default” one in system. This one comes from the httpcore module and it lets us search for a given string within the HttpHeaderValues type. 

Note: Status code 302 seems to be a unique case, so that is why we have an `or` statement on that line.

And there we have it! That is everything we need to know to create a simple directory brute force tool in Nim!
