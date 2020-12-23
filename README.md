# weather-ab
![](https://github.com/ganochenkodg/weather-ab/blob/main/screenshot.png)

### description
Application for comparing the archive of weather in different cities of the world. Indispensable for people thinking about migration.
Used tools:

[Weather API](https://www.visualcrossing.com/weather-api) - api for receiving archived weather records. Provides 1000 entries per day for free.

[Google places API](https://developers.google.com/places/web-service/overview) - api for auto-completion of addresses, used for auto-completion of city names.

[Svelte](https://svelte.dev/) - reactive JS framework.

[CanvasJS](https://canvasjs.com/) - responsive HTML5 charting library with a simple API.

### requirements

1. First of all you need API KEY for getting weather archive. Go [here](https://www.visualcrossing.com/weather/weather-data-services) and register. Put a key to [env.js](https://github.com/ganochenkodg/weather-ab/blob/main/src/env.js).
2. [Get](https://developers.google.com/maps/premium/apikey/places-apikey) an API key for google places and write it to the [env.js](https://github.com/ganochenkodg/weather-ab/blob/main/src/env.js) too.

### usage

1. install all project dependencies and run app in dev mode

```
npm install && npm run dev
```
2. Open [http://localhost:5000](http://localhost:5000)...

3. Also it have Dockerfile for dockerize your app.

```
docker build -t weather-ab ./
docker run -p 80:80 weather-ab
```
