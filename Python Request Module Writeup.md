1. The requests module in Python is the de facto standard for making HTTP requests in Python.
2. To import requests module, use `import requests`
3. To send HTTP request, 
```python
response = requests.get('https://google.com')
response = requests.post('https://google.com') # send post request
response = requests.put('https://google.com') # send put request
response = requests.head('https://google.com') # send head request
response = requests.options('https://google.com') # send options request
```
- All these methods will return a response object `<class 'requests.models.Response'>`
4. To get the status code of the request,
```python
>>> response.status_code
200
```
5. Response object will be evaluated to True if the `obj.status_code` is smaller than 400. Else, it returns False.
```PYTHON
>>> response.status_code
200
>>> if response:
...     print('Sucess')
... else:
...     print('Fail')
... 
Sucess
```
6. To get contents of a response,
```python
>>> response.content
b'<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage...</body></html>'
```
- Note that the content is in byte string
7. We can use `.text` to convert the byte string into string.
```python
>>> response.text
'<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage...</body></html>'
```
- Requests will try to guess the type of encoding (UTF-8 etc) based on the HTTP header
8. If the HTTP response content is a json, we can convert the content into a Python Dictionary (json)
```python
>>> response = requests.get('https://pokeapi.co/api/v2/pokemon/pikachu')
>>> respContent = response.json()
>>> respContent.keys()
dict_keys(['abilities', 'base_experience', 'cries', 'forms', 'game_indices', 'height', 'held_items', 'id', 'is_default', 'location_area_encounters', 'moves', 'name', 'order', 'past_abilities', 'past_types', 'species', 'sprites', 'stats', 'types', 'weight'])
```

# Headers
1. To access the HTTP headers,
```python
>>> response.headers
{'Date': 'Fri, 12 Jul 2024 06:34:42 GMT', 'Content-Type': 'application/json; charset=utf-8', 'Content-Length': '6958', 'Connection': 'keep-alive', ... 'Server': 'cloudflare', 'CF-RAY': '8a1f024f5fc581c8-SIN'}
```
- It returns a dictionary(-like) object.
2. To access a certain header, access it like a normal dictionary
```python
>>> response.headers['Content-Length']
'6958'
>>> response.headers['conTent-leNgth'] # case is not important (Because HTTP specification defines headers as case-insensitive)
'6958'
```

# Query Parameters
## General
1. To customise Request Headers,
```python
response = requests.get(
	'https://httpbin.org/',
	params = b'a=doSomething&b=secondParameter',
	headers = {'Accept' : 'application/json'},
)
```
2. To disable SSL certificate verification,
```python
requests.get("https://api.github.com", verify=False)
```
## GET parameters
1. To specify GET parameters, 
```python
response = requests.get(
	'https://websec.fr/level10/index.php',
	params = b'a=doSomething&b=secondParameter'
)

# second method
response = requests.get(
	'https://websec.fr/level10/index.php',
	params = {'a':'doSomething', 'b':'secondParameter'}
)
```
- This will send a GET request to `https://websec.fr/level10/index.php?a=doSomething&b=secondParameter`
## POST parameters/ Message body
1. To specify the JSON contents of the message body,
```python
>>> response = requests.post(
...     'https://httpbin.org/',
...     json = {'hello' : 'world'},
... ) # json
```
- Note that the `Content-Type` header will be `application/json`

2. To specify `x-www-form-urlencoded` contents of the message body (default),
```python 
response = requests.post(
	'http://127.0.0.1:8000/',
    data = {'hello' : 'world'},
) # json

print(response.request.body)
# hello=world
```
# Proxy
1. We can proxy our request through another machine
```python
response = requests.post(
	'https://websec.fr/level10/index.php', # target URL
	data = {'f' : 'index.php', 'hash' : 'b4382d64'}, #f=index.php&hash=b4382d64
	proxies = {
		'https' : 'http://127.0.0.1:8080', # Specify HTTPS proxy and send it through a certain port
	},
	verify= False, # Trust Burpsuite's self signed CA certificate
)
```
# Redirect
1. To disable redirects,
```python
res = session.post(
    BASE_URL+'/submit/helloinput'
    allow_redirects = False
)
```
# Sessions
1. To manage cookies and other forms of persistence,
```js
session = requests.Session()
```
