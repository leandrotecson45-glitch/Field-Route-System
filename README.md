
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Field Route System - Real Road Route</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<link rel="stylesheet" href="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.css"/>

<style>
body{
  margin:0;
  font-family:Arial,Helvetica,sans-serif;
  background:#f4f6f8;
}
.top{
  background:#fff;
  padding:12px;
  box-shadow:0 2px 10px rgba(0,0,0,.08);
}
h2{
  margin:0 0 10px;
  font-size:20px;
}
textarea,input,button{
  width:100%;
  box-sizing:border-box;
  padding:10px;
  margin-top:6px;
  border:1px solid #ccc;
  border-radius:8px;
  font-size:14px;
}
button{
  background:#0d6efd;
  color:#fff;
  border:none;
  font-weight:bold;
  cursor:pointer;
}
button:hover{opacity:.92;}
#map{
  height:62vh;
}
.result{
  margin-top:10px;
  background:#eef5ff;
  padding:10px;
  border-radius:8px;
  font-weight:bold;
  line-height:1.6;
}
.small{
  font-size:12px;
  color:#666;
  margin-top:6px;
}
</style>
</head>

<body>

<div class="top">
  <h2>🛣️ Field Route System (Real Road Route)</h2>

  <textarea id="input" rows="5">Lat: 15.75000 | Lon: 120.96000
Lat: 15.76000 | Lon: 120.97000
Lat: 15.77000 | Lon: 120.98000</textarea>

  <input type="number" id="rate" value="10" placeholder="Rate per KM">

  <button onclick="generateRoute()">Generate Real Road Route</button>

  <div class="result" id="result">
    Waiting for route...
  </div>

  <div class="small">
    Accurate road-following route using OpenStreetMap + OSRM (free)
  </div>
</div>

<div id="map"></div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet-routing-machine/dist/leaflet-routing-machine.js"></script>

<script>
let map = L.map('map').setView([15.75,120.96],12);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
  attribution:'© OpenStreetMap'
}).addTo(map);

let routeControl = null;

function parseInput(){
  const lines = document.getElementById("input").value.trim().split("\n");
  let points = [];

  lines.forEach(line=>{
    let m = line.match(/Lat:\s*([0-9.\-]+)\s*\|\s*Lon:\s*([0-9.\-]+)/i);

    if(m){
      points.push(
        L.latLng(
          parseFloat(m[1]),
          parseFloat(m[2])
        )
      );
    }
  });

  return points;
}

function generateRoute(){

  let points = parseInput();

  if(points.length < 2){
    alert("Need at least 2 coordinates.");
    return;
  }

  if(routeControl){
    map.removeControl(routeControl);
  }

  routeControl = L.Routing.control({
    waypoints: points,
    addWaypoints:false,
    routeWhileDragging:false,
    draggableWaypoints:false,
    fitSelectedRoutes:true,
    show:false,

    lineOptions:{
      styles:[
        {color:"blue", weight:6, opacity:0.9}
      ]
    },

    createMarker:function(i,wp){
      return L.marker(wp.latLng).bindPopup("Point " + (i+1));
    },

    router:L.Routing.osrmv1({
      serviceUrl:"https://router.project-osrm.org/route/v1"
    })

  }).addTo(map);

  routeControl.on("routesfound", function(e){

    let route = e.routes[0];
    let km = route.summary.totalDistance / 1000;

    let rate = parseFloat(document.getElementById("rate").value || 0);
    let payout = km * rate;

    document.getElementById("result").innerHTML =
      "🛣️ Route Status: Real Road Route<br>" +
      "📏 Total KM: " + km.toFixed(2) + "<br>" +
      "💵 Rate/KM: ₱" + rate.toFixed(2) + "<br>" +
      "💰 TOTAL PAYOUT: ₱" + payout.toFixed(2);
  });

  routeControl.on("routingerror", function(){
    alert("Unable to generate route now. Please try again.");
  });
}
</script>

</body>
</html>
