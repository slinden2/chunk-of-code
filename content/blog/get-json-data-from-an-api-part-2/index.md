---
title: "Get JSON data from an API (part 2)"
description: "Helping a guy to fetch some data. Part 2"
published: true
date: "20190225"
tags: ["aiohttp", "API", "asyncio", "concurrency"]
---

In this post I will show you how to use [asyncio](https://docs.python.org/3/library/asyncio.html) module to get JSON data on an API asynchronously. The advantage of `asyncio` is that it makes the processing of IO-bound tasks much faster. IO-bound task is a task equivalent to, for example, waiting input from user or a network operation.

While the program is waiting for an IO bound task to complete it can start other long IO-bound tasks and as a result they will complete almost at the same time. This way the long tasks overlap and in total take less time.

In this case our IO-bound task is getting data from the API. When we the program is waiting for an API call to complete. It can already start other API calls. When all of the API calls are completed, the program will proceed further.

### Asynchronous version

You can download the source file used as an example from [here](http://s000.tinyupload.com/index.php?file_id=05405036425646895562).

Let's start with the main program that runs when the module is called.

```py
import csv
import aiohttp
import asyncio

SOURCE_FILE = business_ids.csv
OUTPUT_FILE = results.csv
URL = http://avoindata.prh.fi/tr/v1/

if __name__ == __main__:

    company_list = get_companies(SOURCE_FILE)
    data = asyncio.get_event_loop().run_until_complete(get_api_data(company_list))

```

First we import the modules that are needed for running the program. You can see that we won't be using [requests](http://docs.python-requests.org/en/master/) like in the [synchronous version](../get-json-data-from-an-api-part-1/). We are using [aiohttp](https://aiohttp.readthedocs.io/en/stable/) instead. This is because `requests` library does not support awaitables. I won't go into detail here, but it means that the library's class constructor doesn't have a `__await__` magic method and there for its methods can't be awaited.

`asyncio` is the module that we are using for the event loop and for the processing of the coroutines. It is used to write concurrent Python code and it is a perfect fit for IO-bound programs. It is more complicated than [threading](https://docs.python.org/3/library/threading.html), but it is quite easy to create simple applications with it.

After importing the necessary libraries, we initialize the constant that we used also in the [synchronous version](/get-json-data-from-an-api-part-1/). Then we create the `company_list` variable by calling `get_companies`. Passing `SOURCE_FILE` is not 100% necessary, but it makes the code more readable and easier to follow.

After receiving the list of companies, it is time to start using the `asyncio` library. First we create an event loop and start it in the main module. `data` will contain the list of results for further processing.

```py
async def get_api_data(company_list):
    tasks = []
    connector = aiohttp.TCPConnector(limit=10)

    async with aiohttp.ClientSession(connector=connector) as session:

        for company in company_list:
            number = company[number]
            code = company[business_id]
            if code[-2] != -:
                code = code[:7] + - + code[-1]

            tasks.append(fetch(session, number, code, URL+code))

        data = await asyncio.gather(*tasks)

    return data
```

This is the function where all the magic happens. `async def` is a special syntax for asynchronous functions.

First we create a list that will contain the tasks for each individual API call and a TCPConnector that is used to limit the maximum amount of simultaneous connections to 10. We do not want to bombard the API too heavily.

Then we use a class from the `aiohttp` library. `ClientSession` takes the previously created connector as an argument. We can use the same session for all API calls. If we used `threading` library, for every thread we would need to create a separate session. With `asyncio` that is not the case.

The for loop is directly from the [synchronous version](/get-json-data-from-an-api-part-1/) . It modifies the business_ids so that every single one of the has a hyphen `-` as the second last character. The API call won't work if the ID is not perfectly formatted.

Then we append the tasks to the task list. Take a look at the `fetch` function below. We pass the session, number, code and url as arguments into the function.

```py
async def fetch(session, number, code, url):
    async with session.get(url) as response:
        return (number, code, await response.json())
```

This part doesn't differ much from the [synchronous version](/get-json-data-from-an-api-part-1/). The main difference is that we can `await` the response while. `await` means that we pause the function and give up control to the event loop until the awaited operation is completed. This way we can await many API calls simulaneously.

The fetching will be done when `await asyncio.gather(*tasks)` is called. When all the responses are completed we return `data`. Now we should have a list of the results of all tasks for further processing. From here we could write the data to a file as we did in the [synchronous version](/get-json-data-from-an-api-part-1/).

This example doesn't work perfectly though. The API doesn't support many API calls at once and some of them will result in 503 error. The scope of this post was to show how to do the API call asyncronously, but unfortunately the API I chose to use wasn't the right one for this purpose.
