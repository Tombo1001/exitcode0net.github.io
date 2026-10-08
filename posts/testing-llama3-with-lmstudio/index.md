# Testing Llama3 With LM Studio

> Taking a look at the the latest open source llama 3 model after its April 2024 release. Testing its performance and accuracy with LM Studio.

Source: https://exitcode0.net/posts/testing-llama3-with-lmstudio/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-04-19
Updated: 2026-10-08
Tags: stable-diffusion, ai, art


MetaAI have [just introduced](https://about.fb.com/news/2024/04/meta-ai-assistant-built-with-llama-3/) Llama 3 to the world and the open source community is already putting it through its paces and pushing to find the limits of what it can produce. One fantastic tool which has made self hosted LLMs that rival paid services like [ChatGPT](https://chat.openai.com/) and [Claude](https://claude.ai/) possible is [LM Studio](https://lmstudio.ai/). With version 0.2.20 they bough about support for llama 3 along with a [GGUF quantized 8 billion parameter](https://huggingface.co/lmstudio-community/Meta-Llama-3-8B-Instruct-GGUF) version of the model.

## Installing the model

The beautiful thing about LM studio is how simple it is to get up and running. This project has made brilliant progress towards bringing local LLMs to the world. I feel that it is of upmost importance that projects like LM studio and Jan continue to grow and prevent services like OpenAI's ChatGPT from building a paywall around the technological revolution that is large language models. There will always be a place for paid version of this technology, but it cannot exists as the only available option.

Installing the new llama3 model is just as simple as ever in LM Studio. Once you have updated to version 0.2.20, you will see the lmstudio-community model on the home page. However, clicking download from here, might not given you the highest quantized version that your system is capable of running. Instead click the search icon and you will be directed to a page which lists all available version of that model.

![Install llama3 model in LM Studio: Finding the model](https://exitcode0.net/images/llama3-lmstudio-install-01.png)

Now you can selected the version that best suits your needs or system profile. I have a GPU with 12GB VRAM and can therefore enable full GPU offload for the Meta-Llama-3-8B-Instruct-Q8_0.gguf (8.54 GB) version of the model. Hit download on the model version you want to grab.

![Install llama3 model in LM Studio: Installing the best model](https://exitcode0.net/images/llama3-lmstudio-install-02.png)

Now head over to the chat tab (on the left) and select the llama 3 model. At this point you might be prompted to accept a new prompt or keep your existing prompt. You want to accept the new prompt.

![Install llama3 model in LM Studio: Loading the model](https://exitcode0.net/images/llama3-lmstudio-install-03.png)

## Default llama3 prompt

Our default llama3 prompt in LM Studio looks like this:

```json
{
  "name": "Llama 3",
  "inference_params": {
    "input_prefix": "<|start_header_id|>user<|end_header_id|>\n\n",
    "input_suffix": "<|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n",
    "pre_prompt": "You are a helpful, smart, kind, and efficient AI assistant. You always fulfill the user's requests to the best of your ability.",
    "pre_prompt_prefix": "<|start_header_id|>system<|end_header_id|>\n\n",
    "pre_prompt_suffix": "<|eot_id|>",
    "antiprompt": ["<|start_header_id|>", "<|eot_id|>"]
  }
}
```

I chose to set full GPU utilisation, keep the entire model in RAM and bumped the context length all the way up to the support maximum: 8192.

![Prompt configuration for llama3 model in LM Studio](https://exitcode0.net/images/llama3-lmstudio-prompt.png)

As I will discus in the upcoming performance section, I did hit the context limit, which ultimately lead to some less satisfactory output.

## Performance

Let me start by providing some system specs as context. I have by 2024 standards, a modest desktop computer:

- Ryzen 5600 6c/12t CPU
- 32GB 3200MHz DDR4 RAM
- Nvidia RTX 3060 12GB GDDR6 GPU

It would be great to test these models on bleeding edge hardware, enterprise hardware and even Apple M3 silicon, but hardware isn't cheap these days and I, like many, don't have the resources to acquire it. Thankfully it seems that llama3 performance at this hardware level is very good and there's minimal, perceivable slowdown as the context token count increases. I was experiencing speeds of 23 tokens per second in LM Studio and my chat focusing on writing a python script was remarkable.

![Performance of llama3 model in LM Studio](https://exitcode0.net/images/llama3-lmstudio-performance.png)

## Results

Ok so, what did I _ask_ llama3? This version of the model is fine tuned for instructional tasks, so for now, I asked for help improving a basic python script. This is a great way to test how current and accurate a model is. Here is my full transcript...

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

I'd be happy to help you improve your code. Here is a refactored version of your code using functions and threading:

```python
import requests
import threading
import time
import psycopg2

# Configuration variables
TIME_BETWEEN_REQUESTS = 5  # in seconds
AUTHORIZATION_KEY = "SOMERANDOMAPIKEY"

class Trading212API:
    def __init__(self, authorization_key):
        self.authorization_key = authorization_key

    def get_pies(self):
        url = "https://demo.rading212.com/api/v0/equity/pies"
        headers = {"Authorization": self.authorization_key}
        response = requests.get(url, headers=headers)
        return response.json()

class DatabaseWriter:
    def __init__(self, database_url, table_name):
        self.database_url = database_url
        self.table_name = table_name

    def write_data_to_db(self, data):
        conn = psycopg2.connect(self.database_url)
        cur = conn.cursor()
        for item in data:
            cur.execute("INSERT INTO {} (id, cash, dividend_details, result, progress, status)".format(self.table_name),
                        (item['id'], item['cash'], item['dividendDetails'], item['result'], item['progress'], item['status']))
        conn.commit()
        conn.close()

def send_request_and_write_to_db(api, db_writer):
    while True:
        data = api.get_pies()
        if data:
            db_writer.write_data_to_db(data)
        time.sleep(TIME_BETWEEN_REQUESTS)

if __name__ == "__main__":
    api = Trading212API(AUTHORIZATION_KEY)
    db_writer = DatabaseWriter("host=localhost port=5432 dbname=mydatabase user=myuser password=mypassword", "mytable")
    thread = threading.Thread(target=send_request_and_write_to_db, args=(api, db_writer))
    thread.start()
```

Here's what I did:

1. Extracted the API key and other configuration variables to separate constants.
2. Created a `Trading212API` class that encapsulates the API request logic.
3. Created a `DatabaseWriter` class that encapsulates the database writing logic.
4. Introduced a `send_request_and_write_to_db` function that sends an API request and writes the response to the database. This function is run in a separate thread using `threading`.
5. In the main code, I create instances of the `Trading212API` and `DatabaseWriter` classes and start the thread.

Note: You'll need to install the `psycopg2` library to use this code with PostgreSQL. Also, make sure to replace the database URL and table name in the `DatabaseWriter` constructor with your actual values.

Also, you may want to consider handling errors and exceptions more robustly in the code. This is just a starting point, and you may need to add additional error handling depending on your specific requirements.

---

**As this will be running in a container, TIME_BETWEEN_REQUESTS and AUTHORIZATION_KEY will be environment variables, can you adjust the code to load them from env?**

---

Yes, I can help with that! Here's an updated version of the code that loads the `TIME_BETWEEN_REQUESTS` and `AUTHORIZATION_KEY` from environment variables:

```python
import requests
import threading
import time
import psycopg2
from os import getenv

# Configuration variables
TIME_BETWEEN_REQUESTS = int(getenv("TIME_BETWEEN_REQUESTS", 5))  # in seconds
AUTHORIZATION_KEY = getenv("AUTHORIZATION_KEY")

class Trading212API:
    def __init__(self, authorization_key):
        self.authorization_key = authorization_key

    def get_pies(self):
        url = "https://demo.rading212.com/api/v0/equity/pies"
        headers = {"Authorization": self.authorization_key}
        response = requests.get(url, headers=headers)
        return response.json()

class DatabaseWriter:
    def __init__(self, database_url, table_name):
        self.database_url = database_url
        self.table_name = table_name

    def write_data_to_db(self, data):
        conn = psycopg2.connect(self.database_url)
        cur = conn.cursor()
        for item in data:
            cur.execute("INSERT INTO {} (id, cash, dividend_details, result, progress, status)".format(self.table_name),
                         (item['id'], item['cash'], str(item['dividendDetails']), str(item['result']), item['progress'], item['status']))
        conn.commit()
        conn.close()

def send_request_and_write_to_db(api, db_writer):
    while True:
        data = api.get_pies()
        if data:
            db_writer.write_data_to_db(data)
        time.sleep(TIME_BETWEEN_REQUESTS)

if __name__ == "__main__":
    api = Trading212API(AUTHORIZATION_KEY)
    db_writer = DatabaseWriter(getenv("DATABASE_URL"), getenv("TABLE_NAME"))
    thread = threading.Thread(target=send_request_and_write_to_db, args=(api, db_writer))
    thread.start()
```

In this code:

- We've used the `getenv` function from Python's `os` module to load the environment variables `TIME_BETWEEN_REQUESTS` and `AUTHORIZATION_KEY`. The default value for `TIME_BETWEEN_REQUESTS` is 5 seconds if it's not provided.
- For loading database URL and table name, we're using `getenv` function in the same way as above. Make sure to set these environment variables in your Docker container before running this script.

By doing so, you can easily manage these settings from your environment or configuration files without having to modify the code itself.

---

**I will be using postgres as a source for grafana, please can you include the current time in the postgres INSERT command?**

---

Here's an updated version of the `write_data_to_db` method that includes the current timestamp in the INSERT command:

```python
class DatabaseWriter:
    def __init__(self, database_url, table_name):
        self.database_url = database_url
        self.table_name = table_name

    def write_data_to_db(self, data):
        conn = psycopg2.connect(self.database_url)
        cur = conn.cursor()
        for item in data:
            now = time.strftime('%Y-%m-%d %H:%M:%S')  # Get current timestamp
            cur.execute("""
                INSERT INTO {}
                (id, cash, dividend_details, result, progress, status, inserted_at)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
            """.format(self.table_name),
                         (item['id'], item['cash'], str(item['dividendDetails']), str(item['result']), item['progress'], item['status'], now))
        conn.commit()
        conn.close()
```

---

**Can you please give me the full script with all current modifications?**

---

Here is the full script with all the modifications we've made:

```python
import requests
import threading
import time
import psycopg2
from os import getenv, environ
import datetime
environ['TZ'] = 'UTC'  # Set timezone to UTC for timestamp conversion
time.tzset()

# Configuration variables
TIME_BETWEEN_REQUESTS = int(getenv("TIME_BETWEEN_REQUESTS", 5))  # in seconds
AUTHORIZATION_KEY = getenv("AUTHORIZATION_KEY")
DATABASE_URL = getenv("DATABASE_URL")
TABLE_NAME = getenv("TABLE_NAME")

class Trading212API:
    def __init__(self, authorization_key):
        self.authorization_key = authorization_key

    def get_pies(self):
        url = "https://demo.rading212.com/api/v0/equity/pies"
        headers = {"Authorization": self.authorization_key}
        response = requests.get(url, headers=headers)
        return response.json()

class DatabaseWriter:
    def __init__(self, database_url, table_name):
        self.database_url = database_url
        self.table_name = table_name

    def write_data_to_db(self, data):
        conn = psycopg2.connect(self.database_url)
        cur = conn.cursor()
        for item in data:
            now = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')  # Get current timestamp
            cur.execute("""
                INSERT INTO {}
                 (id, cash, dividend_details, result, progress, status, inserted_at)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
             """.format(self.table_name),
                         (item['id'], item['cash'], str(item[ 'dividendDetails']), str(item['result']), item['progress'], item['status'], now))
        conn.commit()
        conn.close()

def send_request_and_write_to_db(api, db_writer):
    while True:
        data = api.get_pies()
        if data:
            db_writer.write_data_to_db(data)
        time.sleep(TIME_BETWEEN_REQUESTS)

if __name__ == "__main__":
    api = Trading212API(AUTHORIZATION_KEY)
    db_writer = DatabaseWriter(DATABASE_URL, TABLE_NAME)
    thread = threading.Thread(target=send_request_and_write_to_db, args=(api, db_writer))
    thread.start()
```

This script should now work as expected. It loads the configuration variables from environment variables, makes API requests to the Trading212 API, writes the results to a PostgreSQL database with the current timestamp, and runs in an infinite loop with the specified interval between requests.

---

**I am getting this error because I am developing on windows:
`AttributeError: module 'time' has no attribute 'tzset'`.
Is there an alternative way to do this?**

---

Yes, you're correct that `time.tzset()` is not available in Windows. The `tzset` function is a Unix-specific function that sets the timezone environment variable.

To set the timezone on Windows, you can use the `os.environ['TZ'] = 'UTC'` line before calling `time.tzset()`. However, since `time.tzset()` is not available in Windows, this line will have no effect.

So, if you're developing on Windows, you can simply remove the lines that set the timezone and use the `datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')` to get the current timestamp. This will give you the local system time without considering any timezone settings.

Here is the modified code:

```python
import requests
import threading
import time
import datetime
import psycopg2
from os import getenv, environ

# Configuration variables
TIME_BETWEEN_REQUESTS = int(getenv("TIME_BETWEEN_REQUESTS", 5))  # in seconds
AUTHORIZATION_KEY = getenv("AUTHORIZATION_KEY")
DATABASE_URL = getenv("DATABASE_URL")
TABLE_NAME = getenv("TABLE_NAME")

class Trading212API:
    def __init__(self, authorization_key):
        self.authorization_key = authorization_key

    def get_pies(self):
        url = "https://demo.rading212.com/api/v0/equity/pies"
        headers = {"Authorization": self.authorization_key}
        response = requests.get(url, headers=headers)
        return response.json()

class DatabaseWriter:
    def __init__(self, database_url, table_name):
        self.database_url = database_url
        self.table_name = table_name

    def write_data_to_db(self, data):
        conn = psycopg2.connect(self(database_url))
        cur = conn.cursor()
        for item in data:
            now = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')  # Get current timestamp
            cur.execute("""
                INSERT INTO {}
                 (id, cash, dividend_details, result, progress, status, inserted_at)
                VALUES (%s, %s, %s, %s, %s, %s, %s)
             """ .format(self.table_name),
                          (item['id'], item['cash'], str(item[ 'dividendDetails']), str(item[ 'result']), item['progress'], item['status'], now))
        conn.commit()
        conn.close()

def send_request_and_write_to_db(api, db_writer):
    while True:
        data = api.get_pies()
        if data:
            db_writer.write_data_to_db(data)
        time.sleep(TIME_BETWEEN_REQUESTS)

if __name__ == "__main__":
    api = Trading212API(AUTHORIZATION_KEY)
    db_writer = DatabaseWriter(DATABASE_URL, TABLE_NAME)
    thread = threading.Thread(target=send_request_and_write_to_db, args=(api, db_writer))
    thread.start()
```

This code should work as expected on both Unix and Windows.

---

**One last task, can you help me write a docker compose file which will create a container to run our python code, a container for postgres (where our python script will insert to) including an init script to make the table, and a grafana instance? Can you include any environment variables which we will need to pass to our python script such as database_url and table_name**

---

Here is a Docker Compose file that creates three containers: one for your Python script, one for PostgreSQL with an initial script to create the table, and one for Grafana:

```yaml
version: "3"

services:
  db:
    image: postgres
    restart: always
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydatabase
    volumes:
      - db-data:/var/lib/postgresql/data

  init-db:
    build: ./init-db
    depends_on:
      - db
    command: python init.py

  python-app:
    build: .
    depends_on:
      - db
    environment:
      DATABASE_URL: "postgresql://myuser:mypassword@db:5432/mydatabase"
      TABLE_NAME: "mytable"
      TIME_BETWEEN_REQUESTS: 5
      AUTHORIZATION_KEY: "your-authorization-key"
    command: python app.py

  grafana:
    image: grafana/grafana
    restart: always
    ports:
      - "3000:3000"

volumes:
  db-data:
```

Here's what this file does:

1. It creates a `db` service that runs the official PostgreSQL image, sets environment variables for the user, password, and database name, and mounts a volume to persist data.
2. It creates an `init-db` service that builds from a directory named `init-db` in the current directory, depends on the `db` service, and runs a command to create the table using a Python script named `init.py`.
3. It creates a `python-app` service that builds from the current directory, depends on the `db` service, sets environment variables for the database URL, table name, time between requests, and authorization key, and runs the command to start the Python app.
4. It creates a `grafana` service that runs the official Grafana image, exposes port 3000, and restarts always.

To use this file, create a directory named `init-db` in the same directory as your Docker Compose file, and add an `init.py` script to it with the following contents:

```python
import psycopg2

conn = psycopg2.connect(
    host="localhost",
    database="mydatabase",
    user="myuser",
    password="mypassword"
)
cur = conn.cursor()
cur.execute("""
    CREATE TABLE IF NOT EXISTS mytable (
        id SERIAL PRIMARY KEY,
        cash REAL,
        dividend_details TEXT,
        result REAL,
        progress REAL,
        status TEXT,
        inserted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
""")
conn.commit()
```

This script connects to the PostgreSQL database, creates a table named `mytable` with the specified columns if it doesn't exist, and commits the changes.

Then, you can run `docker-compose up -d` in your terminal to start all services in detached mode.

---

## Conclusion

The responses are very fast and accurate, the model takes care to consider all aspects of the prompt and for the most part holds onto the context of the conversation. When I pivoted towards asking about Docker configurations, things started to fall apart and mistakes were introduced. For example our postgres init script was never going to run without a Dockerfile and the practice of splitting this out into a stand alone services makes the deployment less idempotent. The robots won't be taking out jobs just yet...

![They didn't take our jerbs... yet](https://exitcode0.net/images/sad-robot.png)

Ultimately, I reach the token count and the responses started to drop previously fixed sections of code. This is a common issue with all LLM services and one that Clause 3 looks to solve with enormous context token values... unfortunately this could get very expensive due to the token based pricing structure.

The people of the LM Studio discord server are begging for RAG type document referencing features, which would be great. But, relatively small context token counts will mean that this feature could quickly lead to model confusion.

### TLDR

**Llama3 is another step in the right direction and LM Studio once again proves to be an exceptional demonstration of free open source software, but there is always more work to be done**


---
Markdown version of https://exitcode0.net/posts/testing-llama3-with-lmstudio/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
