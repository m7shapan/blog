---
layout: post
title:  "Golang how to unmarshal a subset of nested JSON data"
tags: golang json parser unmarshaler
---

### Have you ever face a case when you need to get nested data from a JSON object?

I think we face this scenario all the time when we deal with third-party APIs

regularly we use JSON unmarshaling to a struct that contains another struct was the only obvious way, 

__But__ today I'm happy to introduce a new option with NJSON package, an option that gives us the ability and flexibility to unmarshal any nested JSON without creating nested objects, just by it's JSON path

### For example 
we are trying to unmarshal this JSON  string 
```json
{
    "coord": {
        "lon": -0.13,
        "lat": 51.51
    },
    "weather": [
        {
            "id": 300,
            "main": "Drizzle",
            "description": "light intensity drizzle",
            "icon": "09d"
        }
    ],
    "base": "stations",
    "main": {
        "temp": 280.32,
        "pressure": 1012,
        "humidity": 81,
        "temp_min": 279.15,
        "temp_max": 281.15
    },
    "visibility": 10000,
    "wind": {
        "speed": 4.1,
        "deg": 80
    },
    "clouds": {
        "all": 90
    },
    "dt": 1485789600,
    "sys": {
        "type": 1,
        "id": 5091,
        "message": 0.0103,
        "country": "GB",
        "sunrise": 1485762037,
        "sunset": 1485794875
    },
    "id": 2643743,
    "name": "London",
    "cod": 200
}
```
_(JSON from [openweathermap](https://samples.openweathermap.org/data/2.5/weather?q=London,uk&appid=439d4b804bc8187953eb36d2a8c26a02))_

<br />

##### To weather overview struct like this

```go
type Weather struct {
    Location       string  
    Weather        string  
    Description    string  
    Temperature    float32 
    MinTemperature float32 
    MaxTemperature float32 
}
```

<br />
Using the standard JSON package we are going to unmarshal it then restructure it like this

```go
    type Weather struct {
		Location       string
		Weather        string
		Description    string
		Temperature    float32
		MinTemperature float32
		MaxTemperature float32
	}

	type TmpWeather struct {
		Location string `json:"name"`
		Weather  []struct {
			Weather     string `json:"main"`
			Description string `json:"description"`
		} `json:"weather"`
		Temperature struct {
			Temperature    float32 `json:"temp"`
			MinTemperature float32 `json:"temp_min"`
			MaxTemperature float32 `json:"temp_max"`
		} `json:"main"`
	}

	var tmpW TmpWeather
	err := json.Unmarshal([]byte(jsonString), &tmpW)
	if err != nil {
		panic(err)
	}

    fmt.Printf("%+v\n", tmpW)
    // {Location:London Weather:[{Weather:Drizzle Description:light intensity drizzle}] Temperature:{Temperature:280.32 MinTemperature:279.15 MaxTemperature:281.15}}

	weather := Weather{
		Location:       tmpW.Location,
		Weather:        tmpW.Weather[0].Weather,
		Description:    tmpW.Weather[0].Description,
		Temperature:    tmpW.Temperature.Temperature,
		MinTemperature: tmpW.Temperature.MinTemperature,
		MaxTemperature: tmpW.Temperature.MaxTemperature,
	}

    fmt.Printf("%+v\n", weather)
    // {Location:London Weather:Drizzle Description:light intensity drizzle Temperature:280.32 MinTemperature:279.15 MaxTemperature:281.15}
```
<br />

With [NJSON](https://github.com/m7shapan/njson) all we need to do is just add `njson` tag to struct
then unmarshal it using NJSON like this

```go
    type Weather struct {
		Location       string  `njson:"name"`
		Weather        string  `njson:"weather.0.main"`
		Description    string  `njson:"weather.0.description"`
		Temperature    float32 `njson:"main.temp"`
		MinTemperature float32 `njson:"main.temp_min"`
		MaxTemperature float32 `njson:"main.temp_max"`
	}

	var weather Weather
	err := njson.Unmarshal([]byte(jsonString), &weather)
	if err != nil {
		panic(err)
	}

    fmt.Printf("%+v\n", weather)
    // {Location:London Weather:Drizzle Description:light intensity drizzle Temperature:280.32 MinTemperature:279.15 MaxTemperature:281.15}
```

<br />

__Wrapping up__
I hope you like NJSON, for more details you can visit [NJSON GitHub repo](https://github.com/m7shapan/njson), you can find the complete example [Here](https://gist.github.com/m7shapan/e00cb6641f5ee7a468485bf8e89638b1) and for any feedback don't hesitate to contact me on twitter [@m7shapan](https://twitter.com/M7Shapan)


