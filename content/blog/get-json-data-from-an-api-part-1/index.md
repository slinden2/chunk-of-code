---
title: "Get JSON data from an API (part 1)"
description: "Helping a guy to fetch some data."
published: true
date: "20190219"
tags: ["API", "JSON", "requests"]
---

I was reading the programming section of my favorite forum one day and I found a small request written by a fellow forum user.

His problem was that he had a huge list of business ID's and business names, but the names weren't updated perfectly and they were were written and formatted by many people so they weren't as homogenous as one could hope for.

He asked if someone had a program or a script that would download business names from [The Business Information System (Finland)](https://www.ytj.fi/en/index.html). As he already had a list of valid business ID's, I thought that this makes the perfect first post for my new blog so I offered my help.

He sent me the file and I started coding a program that could help him with his task. First I though that a synchronous approach would be enough in this case, but the requests from the BIS were taking a painful amount of time, so I wrote the same thing also asynchronously. Please take a look at the synchronous version below. You can download the example source file [here](http://s000.tinyupload.com/index.php?file_id=05405036425646895562). For the synchronous version you may want to use a smaller source data, because the source file contains 800 lines and doing it synchronously is very slow. I will write another post for the [asynchronous version](/get-json-data-from-an-api-part-2/) later.

### Synchronous version

The modules needed for the syncronous version are requests and csv. Let's import them first and initialize some constant variables:

```py
import requests
import csv

SOURCE_FILE = business_id_source.csv
OUTPUT_FILE = business_id_results.csv
URL = http://avoindata.prh.fi/tr/v1/
```

In this case the data comes from a Finnish website so don't worry about the puzzling URL. The data itself will be in English, so anyone will be able to follow along. The URL results in a bad request. It is just a base URL. In order to get some data, we need to attach a business ID at the end of it. We'll do it later.

```py
result_list = []
company_list = []

# get company codes from the source file
with open(SOURCE_FILE, 'r') as source_file:
    reader = csv.DictReader(source_file, delimiter=,)

    for company in reader:
        company_list.append(company)
```

Here we initialize two lists: One for results and one for company data. Then we open the _source_file_ in a context manager and create an instance of the _csv.DictReader_ class. The delimiter in the source file is a comma, so we specify it also in the parameters. _csv.DictReader_ creates a dictionary out of every row in the _source_file_. The keys of the dictionary will be the same as the field names in the _source_file_.

After that we just iterate over the _reader_ variable and append every dictionary to the _company_list._

```py
# open the output file
with open(OUTPUT_FILE, 'a') as out_file:

    for company in company_list:
        number = company[number]
        business_id = company[business_id]

        # add a hyphen if it is missing
        if business_id[-2] != -:
            business_id = business_id[:7] + - + business_id[-1]

        # get data from the api and save it to file
        with requests.get(URL + business_id) as response:
            if response.ok:
                json_data = response.json()
                name = json_data[results][0][name]
                print(f{number},{business_id},{name}, file=out_file)
            else:
                print(f{number},{business_id},Not Found, file=out_file)
```

This is the last phase of the program. We open the _out_file_ and start the iteration over all the companies in the _company_list_ list. At first, we store numbers and business ID's to variables, but the _business_id_ may not be formatted correctly. Take a closer look at the source file. The correct formatting of the business ID is 1234567-8, but some of the ID's are missing the hyphen. The next step is to add the missing hyphen as I do at lines 9-10.

Now we have our source data formatted correctly. We can proceed with the web request. When the we get back an OK response, we parse it to JSON format, store the business name to a variable and print the data to the _out_file_. If we don't get an OK response, we still write the relative data to the _out_file_.

You can check the result file [here](http://s000.tinyupload.com/index.php?file_id=41009120829398131281).

I also paste the complete code here below so you can just paste it to your editor for closer inspection.

```py
import requests
import csv

SOURCE_FILE = business_id_source.csv
OUTPUT_FILE = business_id_results.csv
URL = http://avoindata.prh.fi/tr/v1/

result_list = []
company_list = []

# get company codes from the source file
with open(SOURCE_FILE, 'r') as source_file:
    reader = csv.DictReader(source_file, delimiter=,)

    for company in reader:
        company_list.append(company)

# open the output file
with open(OUTPUT_FILE, 'a') as out_file:

    for company in company_list:
        number = company[number]
        business_id = company[business_id]

        # add a hyphen if it is missing
        if business_id[-2] != -:
            business_id = business_id[:7] + - + business_id[-1]

        # get data from the api and save it to file
        with requests.get(URL + business_id) as response:
            if response.ok:
                json_data = response.json()
                name = json_data[results][0][name]
                print(f{number},{business_id},{name}, file=out_file)
            else:
                print(f{number},{business_id},Not Found, file=out_file)
```
