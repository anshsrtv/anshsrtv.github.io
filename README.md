# Temperature Monitor

### Two APIs for the following:
1) Read the data from temperature sensors and  
2) Show temperature data from database.


### Working
1. Create a public Github repository with Readme and .gitignore in Python.
2. Clone this repo to your local machine using 
`git clone <url_of_repo>`
3. Started a django project in the repo using 
`django-admin startproject temp-monitor`
4. Started an application inside the project named temp.
`$ cd temp-monitor`
`$ django-admin startapp temp`
5. Regiter the temp application in INSTALLED_APPS in `temp-monitor/settings.py`
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'temp', # <= here
]
```
6. Created Models inside `temp/models.py`
```sh
from django.db import models

class Temperature(models.Model):
    sensor_id = models.IntegerField() #Stores the Sensor id
    time = models.DateTimeField() #Time when the temperature was detected
    temperature = models.IntegerField(help_text="In degree Celsius scale")
```
7. Add GET API in `temp/views.py`
```
# GET API
def temp_get(request):
    # If both the limits for time are specified.
    if 'from' in request.GET and 'to' in request.GET:
        temp_data= Temperature.objects.filter(
        time__gte=request.GET['from'], 
        time__lte=request.GET['to']
        )
    # If just the lower limit of time is specified.
    elif 'from' in request.GET:
        temp_data= Temperature.objects.filter(
        time__gte=request.GET['from']
        )
    # If just the upper limit of time is specified.
    elif 'to' in request.GET:
        temp_data= Temperature.objects.filter(
        time__lte=request.GET['to']
        )
    # If no limit is specified.
    else:
        temp_data= Temperature.objects.all()
    # Converting data to Json.
    data = serializers.serialize(
    'json', 
    list(temp_data), 
    fields=('sensor_id','temperture','time')
    )
    # Returning the Json Data
    return JsonResponse(data, safe=False)
```
 8. Add POST API in `temp/views.py`
```
# POST API
def temp_post(request):
    # Decoding the Json data
    json_data=json.loads(request.body.decode("utf-8")) 
    try:
    # Creating a Record in database for the received data
        Temperature.objects.create(
            sensor_id=int(json_data["sensor_id"]),
            time=json_data["time"],
            temperature=int(json_data["temperature"])
        )
    # If error occured, print in console.
    except Exception as e:
        print(e)
        return HttpResponseServerError()
    # Return Empty response in case of successful POST request
    return JsonResponse({})
```
9. Setup URLs for the APIs.
```
from django.contrib import admin
from django.urls import path
from temp.views import temp_get, temp_post

urlpatterns = [
    path('admin/', admin.site.urls),
    path('temp/', temp_get), # <= GET API
    path('add_temp', temp_post), # <= POST API
]
```
10. Lastly, Push these changes to the Github Repository.
`git push origin main`
