## Flask Async

Flask 2.0 adds built-in support for async routes, error handlers, before and after request functions and teardown callbacks. Currently supports asyncio only.

Async functions require an event loop to run. When a request comes into an async view, **Flask will spawn an event loop in a thread**, run the view function there, and then returns the result.

As a synchronous WSGI application, each request still ties up one worker. Nonetheless, you can run async code within a view including making multiple concurrent database queries, HTTP requests, etc.

## Implementation

https://testdriven.io/blog/flask-async/

```py
urls = ['https://www.kennedyrecipes.com',
        'https://www.kennedyrecipes.com/breakfast/pancakes/',
        'https://www.kennedyrecipes.com/breakfast/honey_bran_muffins/']

# Helper Functions

async def fetch_url(session, url):
    """Fetch the specified URL using the aiohttp session specified."""
    response = await session.get(url)
    return {'url': response.url, 'status': response.status}


# Routes

@app.route('/async_get_urls_v2')
async def async_get_urls_v2():
    """Asynchronously retrieve the list of URLs."""
    async with ClientSession() as session:
        tasks = []
        for url in urls:
            task = asyncio.create_task(fetch_url(session, url))
            tasks.append(task)
        sites = await asyncio.gather(*tasks)

    # Generate the HTML response
    response = '<h1>URLs:</h1>'
    for site in sites:
        response += f"<p>URL: {site['url']} --- Status Code: {site['status']}</p>"

    return response
```

## Caveats

Writing get_event_loop() inside a view function is NOT thread-safe, and will trigger runtime error as it is not called from the main thread.

From the docs, if the current thread doesn't already have an event loop associated with it, the default policy of the method creates one when called from the main thread, but raises RuntimeError otherwise.

```py
import asyncio
import threading

from flask import Flask, jsonify

print(f"In flask global level: {threading.current_thread().name}")
app = Flask(__name__)

@app.route("/toy", methods=["GET"])
def index():
    print(f"Inside flask function: {threading.current_thread().name}")
    loop = asyncio.get_event_loop() # runtime error
    result = loop.run_until_complete(hello())

    return jsonify({"result": result})


async def hello():
    await asyncio.sleep(1)
    return 1


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=4567, debug=False)

# In flask global level: MainThread
# Inside flask function: Thread-1
```

## Alternatives

Quart, FastAPI, aioflask (development).
