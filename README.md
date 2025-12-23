# Weather-Alerts Workflow
AI workflow to invoke Weather API and send real time email alers

Developed in n8n.io, this is an Agentic Workflow that uses AI to to transform and send Weather alerts in email.


<img width="3046" height="742" alt="image" src="https://github.com/user-attachments/assets/acaba253-16b9-425f-b771-f5293218cd56" />


The workflow contains following nodes:
### 1. Schedule Trigger:
   This is a trigger node that is similar to Cron Job and runs every day at 9:00 AM.
   
### 2. Configure Request Node:
   Enables user to confiure request parameters like API key, City etc.,
   Sample Ouput:
  ```
    [
      {
       "weatherApiKey": "21b21e8096cf46ffb9e24941252112",
        "aqi": "no",
        "city": [
          "Tampa",
          "New Delhi",
          "Dallas",
          "Pittsburgh"
        ]
      }
    ]
```

### 3. AI Transform Request:
   Uses AI to create the previous output in a specific JSON format that is expected by the HTTP endpoint.
   Instruction: Split the json into seperate json objects each with one city
   Sample Request:
   ```
     [
        {
          "weatherApiKey": "123455677788abdcfeffgrrg",
          "aqi": "no",
          "city": "Tampa"
        },
        {
          "weatherApiKey": "123455677788abdcfeffgrrg",
          "aqi": "no",
          "city": "New Delhi"
        },
        {
          "weatherApiKey": "123455677788abdcfeffgrrg",
          "aqi": "no",
          "city": "Dallas"
        },
        {
          "weatherApiKey": "123455677788abdcfeffgrrg",
          "aqi": "no",
          "city": "Pittsburgh"
        }
     ]
```
  
### 4. HTTP Request: 
This is an HTTP node that invoked the endpoint. We use http://api.weatherapi.com/v1/current to get current weather details.
  #### Request URL: 
  http://api.weatherapi.com/v1/current.json?key=123455677788abdcfeffgrrg&q=Tampa&aqi=no

### 5. AI Transform Response: 
This is an AI Transform Node that parses the response and formats it in specified format.
  #### Instruction:
       """Extract location.name as city, current.condition.text as condition, current.temp_f as temp, current.humidity as humidity, 
       current.wind_kph as wind, current.pressure_in as pressure, current.feelslike_f as feels_like,   current.uv as uv, 
       current.heatindex_f as heat_index, current.last_updated as run_at. Output structured data with these fields.
      If condition contains rain, snow, drizzle, storm, thunder, alert_type = precipitation alert
      if temp > 90, heat alert, if temp < 32, frost alert, else None"""
   #### Sample Output:
```
      [
          {
            "city": "Tampa",
            "condition": "Partly cloudy",
            "temp": 69.1,
            "humidity": 75,
            "wind": 14,
            "pressure": 30.31,
            "feels_like": 69.1,
            "uv": 0,
            "heat_index": 66.6,
            "run_at": "2025-12-23 00:00",
            "alert_type": "None"
          },
          {
            "city": "New Delhi",
            "condition": "Fog",
            "temp": 54,
            "humidity": 100,
            "wind": 13.7,
            "pressure": 30.06,
            "feels_like": 51.4,
            "uv": 2.6,
            "heat_index": 73.8,
            "run_at": "2025-12-23 10:30",
            "alert_type": "None"
          }
        ]
  ```
   ### 6. Switch Node: 
  Creates multiple output files based on "condition" field and temperature field for different alert types. "Rain Alert", "Frost Alert", "Heat Alert" and Normal.
   ### 7. Set Node: 
  This node sets field "Alert Type" based on $Switch.context.outputName => Outputfilename
   ### 8. Update Table Node: 
  Connects to Supabase and create new records in table weather_logs with all the fields.
  #### Sample record:
```
          [
            {
              "id": 29,
              "run_at": "2025-12-23T00:00:00",
              "alert_type": null,
              "temperature": 69.1,
              "wind_speed": 14,
              "humidity": 75,
              "condition": "Partly cloudy",
              "city": "Tampa",
              "temperature_unit": null,
              "raw_response": {
                "city": "Tampa",
                "condition": "Partly cloudy",
                "temp": 69.1,
                "humidity": 75,
                "wind": 14,
                "pressure": 30.31,
                "feels_like": 69.1,
                "uv": 0,
                "heat_index": 66.6,
                "run_at": "2025-12-23 00:00",
                "alert_type": "None"
              }
            },
            {
              "id": 30,
              "run_at": "2025-12-23T10:30:00",
              "alert_type": null,
              "temperature": 54,
              "wind_speed": 13.7,
              "humidity": 100,
              "condition": "Fog",
              "city": "New Delhi",
              "temperature_unit": null,
              "raw_response": {
                "city": "New Delhi",
                "condition": "Fog",
                "temp": 54,
                "humidity": 100,
                "wind": 13.7,
                "pressure": 30.06,
                "feels_like": 51.4,
                "uv": 2.6,
                "heat_index": 73.8,
                "run_at": "2025-12-23 10:30",
                "alert_type": "None"
              }
            }
          ]
  ```
  ### 9. Gmail Node: 
  Send email with configured subject and body to recipient.    
  #### Sample Email:
  <img width="1022" height="878" alt="image" src="https://github.com/user-attachments/assets/9a00a579-82a0-42db-ab9b-84a097c8253d" />


