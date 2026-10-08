# Testing Llama3.1 (8B) With LM Studio

> Taking a look at the the latest open source llama 3.1 model after its July 2024 release. Testing its performance and accuracy with GPT4ALL.

Source: https://exitcode0.net/posts/testing-llama3_1-with-gpt4all/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-07-28
Updated: 2026-10-08
Tags: llm, ai, gpt, llama


MetaAI have done it again... [Dropping llama3.1](https://llama.meta.com/) under the (mostly) open source license and I have been putting it through its paces and benchmarking to find out how well it performs to models preiously tested: [Testing Llama3 With LM Studio](http://localhost:1313/posts/testing-llama3-with-lmstudio/). Previously we took a look at llama3 in [LM Studio](https://lmstudio.ai/) which has made self hosted LLMs that rival paid services like [ChatGPT](https://chat.openai.com/) and [Claude](https://claude.ai/) possible; this time we are taking a look at llama through the lend of [GPT4All](https://www.nomic.ai/gpt4all).

Per the Meta llama github model card:

The Meta Llama 3.1 collection of multilingual large language models (LLMs) is a collection of pretrained and instruction tuned generative models in 8B, 70B and 405B sizes (text in/text out). The Llama 3.1 instruction tuned text only models (8B, 70B, 405B) are optimized for multilingual dialogue use cases and outperform many of the available open source and closed chat models on common industry benchmarks.

**Model Architecture:** Llama 3.1 is an auto-regressive language model that uses an optimized transformer architecture. The tuned versions use supervised fine-tuning (SFT) and reinforcement learning with human feedback (RLHF) to align with human preferences for helpfulness and safety.

The power consumption of the Nvidia H100 GPUs used to build this model is slightly staggering and certainly opens up a number of questions around the value per kilowatt a model like llama provideS to commerce and GDP:

|                |                               |                                    |                                                                   |
| -------------- | ----------------------------- | ---------------------------------- | ----------------------------------------------------------------- |
| **Model**      | **Training Time (GPU hours)** | **Training Power Consumption (W)** | **Training Location-Based Greenhouse Gas Emissions (tons CO2eq)** |
| Llama 3.1 8B   | 1.46M                         | 700                                | 420                                                               |
| Llama 3.1 70B  | 7.0M                          | 700                                | 2,040                                                             |
| Llama 3.1 405B | 30.84M                        | 700                                | 8,930                                                             |
| Total          | 39.3M                         |                                    | 11,390                                                            |

## Installing the model

Just like LMStudio, [GPT4All](https://www.nomic.ai/gpt4all) makes running large language models extremely accessible and maintains complete privacy of chat data. One bonus feature of GPT4All is the ability to "chat with your documents" generating a RAG index for files or folders. This populates the prompt with context based on your indexed files, allowing for a GPT conversation that has awareness of your documents. This type of integration could provide significant value for Professionals or Businesses who have privacy concerns regarding data upload to cloud based AI services like ChatGPT.

GPT4All has some excellent [Documentation](https://docs.gpt4all.io/gpt4all_desktop/quickstart.html).

Once you have downloaded the latest installer for you platform (Windows, Mac, Linux), you can go ahead and grab a GGUF model with ease:

![Downloading the Meta llama3 (or 3.1) model in GPT4All](https://exitcode0.net/images/download_llama.png)

Then, you can go ahead and start a chat and select the model you wish to use - it is possible to have multiple models downloaded and select between then... useful if you have models suited to specific tasks:

![Launching a chat window in GPT4All](https://exitcode0.net/images/before_first_chat.png)

## Performance

Let me start by providing some system specs as context. I have by 2024 standards, a modest desktop computer:

- Ryzen 5600 6c/12t CPU - https://amzn.to/3zKpFrg (an absolute steal in 2024!)
- 32GB 3200MHz DDR4 RAM - https://amzn.to/3TLWkno
- Nvidia RTX 3060 12GB GDDR6 GPU - https://amzn.to/3zurgl7

It would be great to test these models on bleeding edge hardware, enterprise hardware and even Apple M3 silicon, but hardware isn't cheap these days and I, like many, don't have the resources to acquire it. Thankfully it seems that llama3 performance at this hardware level is very good and there's minimal, perceivable slowdown as the context token count increases. I was experiencing speeds of 23 tokens per second in LM Studio and my chat focusing on writing a python script was remarkable.

**Using the 8B model, I saw a great throughput of 38tok/s**, which feels equal to or faster in most cases than popular online AI chatbots.

## Results

Ok so, what did I _ask_ llama3.1? This version of the model is fine tuned for instructional tasks, so for now, I asked for help improving a basic python script. This is a great way to test how current and accurate a model is. Here is my full transcript...

**Good morning, I have the following python code that I want to improve:**

```python
import requests

url = "https://demo.rading212.com/api/v0/equity/pies"

headers = {"Authorization": "SOMERANDOMAPIKEY"}

response = requests.get(url, headers=headers)
print(response.status_code)
if response.status_code == 200:
data = response.json()
if data:
print(data)
else:
print(response.status_code)
```

**I want to implement functions and threading so that I can run this code in a docker container. I want to pass important variables like the time between requests and the Authorization key. I also want to introduce a function which sends the data returned by the successful request to a postgres database. Here is what typical json response looks like:**

`[{'id': 696969, 'cash': 0.48, 'dividendDetails': {'gained': 7.94, 'reinvested': 7.41, 'inCash': 0.46}, 'result': {'investedValue': 302.4, 'value': 318.35, 'result': 15.95, 'resultCoef': 0.0527}, 'progress': 0.3986, 'status': None}, {'id': 696968, 'cash': 0.12, 'dividendDetails': {'gained': 0.72, 'reinvested': 0.63, 'inCash': 0.09}, 'result': {'investedValue': 300.14, 'value': 404.68, 'result': 104.54, 'resultCoef': 0.3483}, 'progress': 0.0792, 'status': None}, {'id': 696867, 'cash': 0.55, 'dividendDetails': {'gained': 0.55, 'reinvested': 0.0, 'inCash': 0.55}, 'result': {'investedValue': 249.27, 'value': 278.99, 'result': 29.72, 'resultCoef': 0.1192}, 'progress': None, 'status': None}, {'id': 696868, 'cash': 0.0, 'dividendDetails': {'gained': 0.0, 'reinvested': 0.0, 'inCash': 0.0}, 'result': {'investedValue': 0, 'value': 0.0, 'result': 0.0, 'resultCoef': 0}, 'progress': None, 'status': None}, {'id': 2181613, 'cash': 0.0, 'dividendDetails': {'gained': 0.0, 'reinvested': 0.0, 'inCash': 0.0}, 'result': {'investedValue': 0, 'value': 0, 'result': 0, 'resultCoef': 0}, 'progress': None, 'status': None}]`

---

I can help you improve your code by implementing functions and threading as per your requirements.
Here's an improved version of the code:

```python
import requests
from concurrent.futures import ThreadPoolExecutor
import time
import psycopg2

class EquityAPI:
    def __init__(self, url, headers):
        self.url = url
        self.headers = headers

    def get_equity(self):
        response = requests.get(self.url, headers=self.headers)
        return response.json()

    def send_to_postgres(self, data):
        conn = psycopg2.connect(
            host='your_host',
            database='your_database',
            user='your_user',
            password='your_password'
        )

        cur = conn.cursor()
        for item in data:
            query = "INSERT INTO your_table (id, cash, dividend_details) VALUES (%s, %s, %s)"
            values = (item['id'], item['cash'], str(item['dividendDetails']))
            cur.execute(query, values)

        conn.commit()
        conn.close()

def main():
    url = 'https://demo.rading212.com/api/v0/equity/pies'
    headers = {"Authorization": "SOMERANDOMAPIKEY"}

    api = EquityAPI(url, headers)

    while True:
        data = api.get_equity()

        if len(data) > 0 and data[0]:
            print("Received data:", data)

            # Send to Postgres
            api.send_to_postgres(data)

            time.sleep(10)  # Wait for 10 seconds before next request

if __name__ == "__main__":
    main()
```

In this improved code:

We've created a class EquityAPI that encapsulates the API functionality.

The get_equity() method sends an HTTP GET request to the specified URL and returns the JSON response.

The send_to_postgres() method connects to your Postgres database, inserts data into the table using SQL queries, and then commits and closes the connection.

In the main() function, we create an instance of the EquityAPI class and use a while loop with a 10-second sleep between requests.

Note that you'll need to replace 'your_host', 'your_database', 'your_user', and 'your_password' in the Postgres connection string with your actual database credentials. Also, make sure to update your_table name according to your schema.

---

**As this will be running in a container, TIME_BETWEEN_REQUESTS (time.sleep(10)) and AUTHORIZATION_KEY will be environment variables, can you adjust the code to load them from env?**

---

I've updated the code to use environment variables for TIME_BETWEEN_REQUESTS and AUTHORIZATION_KEY. Here's the revised version:

```python
import requests
from concurrent.futures import ThreadPoolExecutor
import time
import psycopg2
import os

class EquityAPI:
def **init**(self, url):
self.url = url
self.headers = {"Authorization": os.environ.get("AUTHORIZATION_KEY")}

    def get_equity(self):
        response = requests.get(self.url, headers=self.headers)
        return response.json()

    def send_to_postgres(self, data):
        conn = psycopg2.connect(
            host=os.environ.get('POSTGRES_HOST'),
            database=os.environ.get('POSTGRES_DB'),
            user=os.environ.get('POSTGRES_USER'),
            password=os.environ.get('POSTGRES_PASSWORD')
        )

        cur = conn.cursor()
        for item in data:
            query = "INSERT INTO your_table (id, cash, dividend_details) VALUES (%s, %s, %s)"
            values = (item['id'], item['cash'], str(item['dividendDetails']))
            cur.execute(query, values)

        conn.commit()
        conn.close()

def main():
url = 'https://demo.rading212.com/api/v0/equity/pies'

    api = EquityAPI(url)

    while True:
        data = api.get_equity()

        if len(data) > 0 and data[0]:
            print("Received data:", data)

            # Send to Postgres
            api.send_to_postgres(data)

            time.sleep(int(os.environ.get('TIME_BETWEEN_REQUESTS')))  # Wait for TIME_BETWEEN_REQUESTS seconds before next request

if **name** == "**main**":
main()
```

In this revised code:

We've removed the headers parameter from the EquityAPI constructor and instead use os.environ.get("AUTHORIZATION_KEY") to load the authorization key from an environment variable.

Similarly, we're using time.sleep(int(os.environ.get('TIME_BETWEEN_REQUESTS'))) to wait for the specified time between requests.

Make sure to set these environment variables in your container's configuration. You can do this by adding a .env file with the following content:

```bash
AUTHORIZATION_KEY=SOMERANDOMAPIKEY
POSTGRES_HOST=your_host
POSTGRES_DB=your_database
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password
TIME_BETWEEN_REQUESTS=10
```

Replace your_host, your_database, your_user, and your_password with your actual Postgres database credentials.

Feel free to ask if there's anything else I can help you improve!

---

**I will be using postgres as a source for grafana, please can you include the current time in the postgres INSERT command?**

---

Here is the revised code that includes the current timestamp in the Postgres insert statement:

```python
import requests
from concurrent.futures import ThreadPoolExecutor
import time
import psycopg2
import os
from datetime import datetime

class EquityAPI:
    def __init__(self, url):
        self.url = url
        self.headers = {"Authorization": os.environ.get("AUTHORIZATION_KEY")}

    def get_equity(self):
        response = requests.get(self.url, headers=self.headers)
        return response.json()

    def send_to_postgres(self, data):
        conn = psycopg2.connect(
            host=os.environ.get('POSTGRES_HOST'),
            database=os.environ.get('POSTGRES_DB'),
            user=os.environ.get('POSTGRES_USER'),
            password=os.environ.get('POSTGRES_PASSWORD')
        )

        cur = conn.cursor()
        for item in data:
            query = "INSERT INTO your_table (id, cash, dividend_details, timestamp) VALUES (%s, %s, %s, %s)"
            values = (item['id'], item['cash'], str(item['dividendDetails']), datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
            cur.execute(query, values)

        conn.commit()
        conn.close()

def main():
    url = 'https://demo.rading212.com/api/v0/equity/pies'

    api = EquityAPI(url)

    while True:
        data = api.get_equity()

        if len(data) > 0 and data[0]:
            print("Received data:", data)

            # Send to Postgres
            api.send_to_postgres(data)

            time.sleep(int(os.environ.get('TIME_BETWEEN_REQUESTS')))  # Wait for TIME_BETWEEN_REQUESTS seconds before next request

if __name__ == "__main__":
    main()
```

In this revised code:

We've added a timestamp column to the Postgres insert statement.

The timestamp is generated using datetime.now().strftime("%Y-%m-%d %H:%M:%S"), which returns the current date and time in the format "YYYY-MM-DD HH:MM:SS".

Make sure your Postgres table has a timestamp column with the correct data type.

---

**One last task, can you help me write a docker compose file which will create a container to run our python code, a container for postgres (where our python script will insert to) including an init script to make the table, and a grafana instance? Can you include any environment variables which we will need to pass to our python script such as database_url and table_name**

---

Here is a Docker Compose file that creates three containers: one for your Python code, another for Postgres with an initial script to create the table, and finally a Grafana instance:

```yaml
version: "3"

services:
  api:
    build: .
    environment:
      - AUTHORIZATION_KEY=SOMERANDOMAPIKEY
      - POSTGRES_HOST=postgres
      - POSTGRES_DB=mydatabase
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - TIME_BETWEEN_REQUESTS=10
    depends_on:
      - postgres

  postgres:
    image: postgres
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - POSTGRES_DB=mydatabase
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  grafana:
    image: grafana/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=grafanapassword
    depends_on:
      - postgres

volumes:
  init.sql:
```

This Docker Compose file:

Creates a container named api that runs your Python code.

Sets the necessary environment variables for your script, including database credentials and time between requests.

Depends on the Postgres service to ensure it's running before starting the API.

Defines another container called postgres, which uses an official PostgreSQL image. It sets up a new user, password, and database name as specified in the environment variables section of this file.

Mounts a volume for your initial SQL script (init.sql) at /docker-entrypoint-initdb.d/init.sql to run it during Postgres initialization.

Here's what you should put inside ./init.sql:

```sql
CREATE TABLE mytable (
  id SERIAL PRIMARY KEY,
  cash DECIMAL(10,2),
  dividend_details JSONB NOT NULL DEFAULT '{}'::JSONB,
  timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

This SQL script creates a table named mytable with the columns you specified in your Python code.

Remember to replace placeholders like SOMERANDOMAPIKEY, mydatabase, myuser, and mypassword with actual values for your application. Also, update any other environment variables as needed according to their usage within your script.

---

**IMPRESSIVE!**

---

## Conclusion

The responses are even faster than llama3 and the responses seem moderately more accurate. In comparison to previous tests, there are some observed improvements when we pivoted from python to docker compose YAML. The indicator has certainly turned slight more towards concern for programmers jobs...

### TLDR

**Llama3.1 another strong increment if you are willing to disregard the insane computer required to train it. THe move toward open source is welcome will hopefully force more innovation and reduce 'moat digging' for paid services**


---
Markdown version of https://exitcode0.net/posts/testing-llama3_1-with-gpt4all/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
