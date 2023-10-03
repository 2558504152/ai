# 
# Digital Assistant Capability Developer Hands On
## Weather: Dialog Functions
- Duration: 15 Minutes
- Goal:
  - In this exercise we'll add the actual API calls to the IBM weather APIs in order to lookup a city entered by the user and then fetching the current weather information.
  - In this part the steps will be described with less details to allow you to apply what you learned in the previous parts. Please refer to the previous excercises if you're unsure on how to create an intent, entity or dialog nodes. 

Some general tips:
- Create the file with file ending `.yaml` in the respective folder.
- Use Ctrl + Space to use the auto-completion provided by the IDE extension.

## Sections
### Weather: Dialog Functions
#### System Alias
#### Dialog Function for looking up location
#### Dialog Function for fetching weather information
#### Adopting the dialog nodes
#### Test it

## Weather: Dialog Functions

### Creating the BTP Destination

In order to make API requests to the IBM weather API, a destination needs to be created in the BTP subaccount which is used for the deployment of the Digital Assistant.

- Name: Weather
- URL: https://api.weather.com/v3
- Authentication: NoAuthentication

Click on "New Property" to add the propery `URL.queries.apiKey` with value `c322ef22435d40bfa2ef22435df0bfbe`.

### System Alias

For calling the weather API of IBM there is already a BTP destination called `WEATHER` maintained in the subaccount. To make use of this destination, open the `capability.sapdas.yaml` file and add the system alias, which can afterwards referred in dialog functions:

```
metadata:
  [...]
  system_aliases:
    WeatherService:
      destination: WEATHER
```

### Dialog Function for looking up location

To organize the dialog functions better, create a folder called `weather` in the `functions` folder. Afterwards create the `lookup_location` and `fetch_weather_info` dialog functions within that folder.

The `lookup_location` dialog function will be used to lookup the city entered by the user to a `placeid`, which is required for quering the weather information. A `city` parameter is required for this dialog function as well as the `api_key`. In the actions of the dialog function, the GET call to the following endpoint should be performed by using the `WeatherService` system alias:

```/location/search?query=<? city ?>&language=en-US&format=json&apiKey=<? api_key ?>```

To get a better idea of the API response returned by the API, the following URL can be either called via Postman or directly in the browser: https://api.weather.com/v3/location/search?query=Walldorf&language=en-US&format=json&apiKey=c322ef22435d40bfa2ef22435df0bfbe

Following fields should be returned in the result of the dialog function (using some SpEL expressions):
- success: boolean to indicate if the lookup was successful
- city: full city details of the city that was found
- placeid: ID required for the weather API representing the city

### Dialog Function for fetching weather information

The `fetch_weather_info` dialog function should require a `city` parameter and execute the following actions:
- a set variable action to set the API key `c322ef22435d40bfa2ef22435df0bfbe` into the local context variable called `api_key`
- a dialog function action to call the previously defined `lookup_location` dialog function
- the actual API call to the weather API (again using the `WeatherService` system alias)

  ```/wx/forecast/daily/3day?placeid=<? weather_location.placeid ?>&units=<? unit ?>&language=en-US&format=json&apiKey=<? api_key ?>```

To get a better idea of the API response returned by the API, the following URL can be either called via Postman or directly in the browser: https://api.weather.com/v3/wx/forecast/daily/3day?placeid=b1acb26e1ddc17dd5ba19c9ef4ec23a6b6ccb3a8c4e67477e6ef984c8bd8dd04&units=m&language=en-US&format=json&apiKey=c322ef22435d40bfa2ef22435df0bfbe

As results of the dialog function the following fields should be returned:
- success: boolean to indicate if the lookup was successful
- minTemp / maxTemp: minimum and maximum temperature in celcius degrees for today
- dayOfWeek: weekday of today
- narrative: description of todays weather

### Adopting the dialog nodes

This dialog function can now be called in the `fetch_weather` dialog node by adding the following before the `finally`:
````
dialog_functions:
  - name: weather/fetch_weather_info
    result_variable: weather_result
    parameters:
      - name: city
        value: "$weather_city"
````

In order to "reset" the city variable in the context every time the weather was displayed and allow requesting the weather information for another city a small addition is required in the `response` section of the `display-weather` dialog node:
````
response:
  context:
    - variable: "weather_city"
      value: null
````

### Test it

Compile and deploy the DA capabilities by pointing to the DA configuration file (e.g. my_first_assistant.da.sapdas.yaml) with the changes (replacing `<userid>` with your i-user):

```
sapdas deploy ../my_first_assistant.da.sapdas.yaml -n <userid>_weather --compile
```
Launch the Digital Assistant via the standalone Web Client:

```
sapdas launch <userid>_weather
```

Play with the assistant. When entering `show weather` (or something similiar - depending on your intent expressions) the DA should ask to enter the city and afterwards show the details on the city found and the weather information for today.
Feel free also to try the other capability asking "is my assistant deployed". The idea is that both capabilities functionalities can be accessed within the same Assistant.
