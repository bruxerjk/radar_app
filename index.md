## Test Radar App 
<head>
  <meta charset="utf-8">
  <style>
    #map {
      width: 80%;
      height: 80%;
    }
    #controller {
      background: #ececec;
      padding: 0.5rem;
	  width: 80%;
    }
    #play,#pause {
      padding: 0rem 1rem;
    }
    #info {
      padding-left: 0.5 rem;
    }
  </style>
  <title>Radar Mosaic</title>
  <meta name="description" content="Radar Mosaic Canada">
  <meta name="author" content="JKB">

  <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.3.0/css/ol.css" type="text/css">
  <link href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet" integrity="sha384-wvfXpqpZZVQGK6TAh5PVlGOfQNHSoD2xbE+QkPxCAFlNEevoEH3Sl0sibVcOQVnN" crossorigin="anonymous">
  <script src="https://cdn.jsdelivr.net/gh/openlayers/openlayers.github.io@master/en/v6.3.0/build/ol.js"></script>
</head>

<body>
  <div id="map">
  </div>
  <div id="controller" role="group" aria-label="Animation controls" style="background: #ececec; padding: 0.5rem;">
    <button id="play" class="btn btn-primary btn-sm" type="button">
            <i class="fa fa-play" style="padding: 0rem 1rem"></i>
    </button>
    <button id="pause" class="btn btn-primary btn-sm" type="button">
      <i class="fa fa-pause" style="padding: 0rem 1rem"></i>
    </button>
    <span id="info" style="padding-left: 0.5rem;"></span>
  </div>

<script>
const parser = new DOMParser();

/* Async function used to retrieve start and end time from RADAR_1KM_RRAI layer GetCapabilities document */
async function getRadarStartEndTime() {
  let response = await fetch(
    "https://geo.weather.gc.ca/geomet/?lang=en&service=WMS&request=GetCapabilities&version=1.3.0&LAYERS=RADAR_1KM_RRAI"
  );
  let data = await response
    .text()
    .then((data) =>
      parser
        .parseFromString(data, "text/xml")
        .getElementsByTagName("Dimension")[0]
        .innerHTML.split("/")
    );
  return [new Date(data[0]), new Date(data[1])];
}

let frameRate = 3.0; // frames per second
let animationId = null;
let startTime = null;
let endTime = null;
let current_time = null;

let layers = [
  new ol.layer.Tile({
    source: new ol.source.XYZ({
		url: 'https://cartodb-basemaps-a.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png'
		})
  }),
  new ol.layer.Image({
    source: new ol.source.ImageWMS({
      format: "image/png",
      url: "https://geo.weather.gc.ca/geomet/",
      params: { LAYERS: "RADAR_1KM_RRAI"},
      transition: 0
    })
  }),
  new ol.layer.Image({
    source: new ol.source.ImageWMS({
      format: "image/png",
      url: "https://geo.weather.gc.ca/geomet/",
      params: { LAYERS: "RADAR_COVERAGE_RRAI.INV"},
      transition: 0
    })
  })
];

let map = new ol.Map({
  target: "map",
  layers: layers,
  view: new ol.View({
    //center: ol.proj.fromLonLat([-93, 48]),
    //zoom: 5
	// SW Ontario
	center: ol.proj.fromLonLat([-80, 43.3]),
    zoom: 7.5
  })
});

function updateInfo(current_time) {
  let el = document.getElementById("info");
  el.innerHTML = `Time / Heure (UTC): ${current_time.toISOString()}`;
}

function setTime() {
  var mins = 10;
  if (current_time === null) {
    current_time = startTime;
  } else if (current_time > endTime) {
    current_time = startTime;
  } else {
    new_time = new Date(current_time.getTime() + mins*60000);
    current_time = new_time;
  }
  layers[1]
    .getSource()
    .updateParams({ TIME: current_time.toISOString().split(".")[0] + "Z" });
  layers[2]
    .getSource()
    .updateParams({ TIME: current_time.toISOString().split(".")[0] + "Z" });
  updateInfo(current_time);
}

getRadarStartEndTime().then((data) => {
  startTime = data[0];
  endTime = data[1];
  setTime();
});

let pause = function () {
  if (animationId !== null) {
    window.clearInterval(animationId);
    animationId = null;
  }
};

let play = function () {
  stop();
  animationId = window.setInterval(setTime, 1000 / frameRate);
};

let startButton = document.getElementById("play");
startButton.addEventListener("click", play, false);

let pauseButton = document.getElementById("pause");
pauseButton.addEventListener("click", pause, false);

</script>

</body>
