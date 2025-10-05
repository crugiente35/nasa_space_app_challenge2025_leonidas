# Satellite Constellation Tracker

This is a web application built with **Vue 3** and **Vite**. It allows users to observe the real-time positions of various satellite constellations over the Earth. The update interval for satellite positions can be customized by the user.

## Features

- Real-time visualization of satellite constellations
- Interactive 3D map interface
- Adjustable update interval for satellite data

## How It Works

For real-time satellite visualization, the application uses:
- Satellite data from [Celestrak TLE](https://celestrak.org/NORAD/elements/index.php?FORMAT=tle) for up-to-date satellite positions
- [satellite-js](https://github.com/shashwatak/satellite-js) for satellite propagation calculations
- [CesiumJS](https://cesium.com/platform/cesiumjs/) to display a 3D map of the Earth
- The web interface is built with [Vue 3](https://vuejs.org/)

## Usage

1. Clone the repository.
2. Install dependencies with `npm install`.
3. Run the development server with `npm run dev`.
4. Open the app in your browser to start tracking satellites.
5. After building the project with `npm run build`, a compiled version of the website will be available in the `./dist` directory. You can upload the contents of this folder to any web server and use the application immediately.

## License

This project is licensed under MIT License.
