<template>
  <div id="map"/>
  <div class="q-pa-md bg-grey-10 text-white">
    <q-slider v-model="year" @change="changeYearListener" :min="1885" :max="2022" :step="1" snap label label-always
              color="red-10" id="slider"/>
  </div>
</template>

<script setup>
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
// import axios from 'axios';
import {onMounted} from 'vue';
import {ref} from 'vue';

import length from '@turf/length';
import along from '@turf/along';

import {geoJSON} from './assets/postcards_subset_geo';

const STYLE_URL = 'mapbox://styles/mikhailmiazin/cl4haqydv001a15p5q2hhf9i8';
const MAPBOX_TOKEN = 'pk.eyJ1IjoibWlraGFpbG1pYXppbiIsImEiOiJjbDNyNWNqOWcxYmg1M2NsMjBpMXE1Y3Z0In0.EKt0NQQJRISdpG0ah6evCw';

// const locationCoords = ref(null);
// const exileCoords = ref(null);
// const postcards = ref(null);


// const API_URL = "http://localhost:8000"

const year = ref(1913);
const geoJSONSource = ref(geoJSON[0]);

// define map handler
let m;

const postcards = [
  {
    'id': 52,
    'url_front': 'https://forum.vgd.ru/file.php?fid=454524&key=1487734059',
    'url_back': 'https://forum.vgd.ru/file.php?fid=454523&key=622039376',
    'text_language': 'русский',
    'date_written': '07.12.1910',
    'date_sent': '07.12.1910',
    'date_received': 'xx.xx.xxxx',
    'address_from': 'Одесса',
    'address_to': 'Кавказ, Тифлис, Авчальская ул., д. 51',
    'country_from': 'Российская империя',
    'country_to': 'Российская империя',
    'settlement_from': 'Херсонская губерния, Одесский уезд, город Одесса',
    'settlement_to': 'Тифлисская губерния, Тифлисский уезд, город Тифлис',
    'name_from': '[отсутствует]',
    'name_to': 'Ол[ьга] Константиновна Карасева',
    'lat_from': '46.4858883',
    'lng_from': '30.683651011',
    'lat_to': '41.6934591',
    'lng_to': '44.8014495',
    'city_to': 'Тифлис',
    'city_from': 'Одесса',
    'time_period': 'Дореволюционные',
    'tags': [
      'Новости',
      'Житейское',
      'Дурные вести',
      'Письма от семьи',
      'Письма от мамы',
      'Шлю привет'
    ]
  },
  {
    'id': 78,
    'url_front': 'https://forum.vgd.ru/file.php?fid=448794&key=1372059731',
    'url_back': 'https://forum.vgd.ru/file.php?fid=448795&key=477877634',
    'text_language': 'русский',
    'date_written': '18.08.1915',
    'date_sent': '[отсутствует]',
    'date_received': '[отсутствует]',
    'address_from': 'Лида Гродненской губернии',
    'address_to': 'г. Рига, Бульвар Наследника, д.3, кв. 16',
    'country_from': 'Российская Империя',
    'country_to': 'Российская Империя',
    'settlement_from': 'Гродненская губерния, Лидский уезд,  город Лида',
    'settlement_to': 'Лифляндская губерния, Рижский уезд, город Рига',
    'name_from': '[нрзб. подпись]',
    'name_to': 'Мария Фельдберг',
    'lat_from': '53.8949515',
    'lng_from': '25.295719',
    'lat_to': '56.9493977',
    'lng_to': '24.1051846',
    'city_to': 'Рига',
    'city_from': 'Лида',
    'time_period': 'Дореволюционные',
    'tags': [
      'Новости',
      'Житейское',
      'Дурные вести',
      'Письма от семьи',
      'Письма от мамы',
      'Шлю привет'
    ]
  },
  {
    'id': 87,
    'url_front': 'https://forum.vgd.ru/file.php?fid=448844&key=988619542',
    'url_back': 'https://forum.vgd.ru/file.php?fid=448845&key=2089344553',
    'text_language': 'русский',
    'date_written': '[отсутствует]',
    'date_sent': '05.04.1914',
    'date_received': 'xx.04.1914',
    'address_from': 'Выру',
    'address_to': 'Ревель, фирма В.И.Горбачев, [? Promenadi], №15',
    'country_from': 'Российская Империя',
    'country_to': 'Российская Империя',
    'settlement_from': 'Лифляндская губерния, Веросский уезд, город Выру',
    'settlement_to': 'Эстляндская губерния, Ревельский уезд, город Ревель',
    'name_from': '[отсутствует]',
    'name_to': 'А.Кейверг',
    'lat_from': '57.8385759',
    'lng_from': '27.0086505',
    'lat_to': '59.4372155',
    'lng_to': '24.7453688',
    'city_to': 'Ревель',
    'city_from': 'Выру',
    'time_period': 'Дореволюционные',
    'tags': [
      'Новости',
      'Житейское',
      'Дурные вести',
      'Письма от семьи',
      'Письма от мамы',
      'Шлю привет'
    ]
  },
];


const itemsFrom = postcards.map(postcard => ({
  'type': 'Feature',
  'geometry': {
    'type': 'Point',
    'coordinates': [+postcard.lng_from, +postcard.lat_from]
  },
  'properties': {
    'city_from': postcard.city_from,
    'city_to': postcard.city_to,
    'url_base': `https://sysblok.ru/postcards/card/${postcard.id}/`,
    'url_front_img': postcard.url_front,
    'destination_lng': +postcard.lng_to,
    'destination_lat': +postcard.lat_to
  }
}));

const itemsTo = postcards.map(postcard => ({
  'type': 'Feature',
  'geometry': {
    'type': 'Point',
    'coordinates': [+postcard.lng_to, +postcard.lat_to]
  },
  'properties': {
    'url_base': `https://sysblok.ru/postcards/card/${postcard.id}/`,
    'url_front_img': postcard.url_front,
    'origin_lng': +postcard.lng_from,
    'origin_lat': +postcard.lat_from
  }
}));

console.log('items from:', itemsFrom);
console.log('items to:', itemsTo);
// const fetchLocations = (year) => {
//   axios.get(`${API_URL}/buildings`)
//     .then((response) => {
//       const homes = response.data.map((res) => ({
//         type: res.home.type,
//         geometry: res.home.geometry,
//         properties: { exile_lon: res.exile.geometry.coordinates[0], exile_lat: res.exile.geometry.coordinates[1], ...res.home.properties }
//       }))
//       const exiles = response.data.map((res) => ({
//         type: res.exile.type,
//         geometry: res.exile.geometry,
//         properties: { home_lon: res.home.geometry.coordinates[0], home_lat: res.home.geometry.coordinates[1], ...res.exile.properties }
//       }))
//       locationCoords.value = homes
//       exileCoords.value = exiles
//     })
//   // response.data.map(case => case.home)
// }

const changeYearListener = (year) => {
  // change USSR borders geoJSON
  let mapBordersId;

  if (year < 1905) {
    mapBordersId = 0;
  } else if (year < 1914) {
    mapBordersId = 1;
  } else if (year < 1917) {
    mapBordersId = 2;
  } else if (year < 1918) {
    mapBordersId = 3;
  } else if (year < 1920) {
    mapBordersId = 4;
  } else if (year < 1921) {
    mapBordersId = 5;
  } else if (year < 1940) {
    mapBordersId = 6;
  } else if (year < 1945) {
    mapBordersId = 7;
  } else if (year < 1991) {
    mapBordersId = 8;
  } else if (year < 2014) {
    mapBordersId = 9;
  } else {
    mapBordersId = 10;
  }

  m.getSource('ussr').setData(geoJSON[mapBordersId]);
};

const calculateArcCoords = (origin, destination, route, steps = 300) => {
  const lineDistance = length(route.features[0]);
  const arc = [];
  for (let i = 0; i < lineDistance; i += lineDistance / steps) {
    const segment = along(route.features[0], i);
    arc.push(segment.geometry.coordinates);
  }
  return arc;
};

const generateArcSource = (origin, destination, steps = 300) => {
  const route = {
    'type': 'FeatureCollection',
    'features': [
      {
        'type': 'Feature',
        'geometry': {
          'type': 'LineString',
          'coordinates': [origin, destination]
        }
      }
    ]
  };
  // calculate coords for arc
  const arc = calculateArcCoords(origin, destination, route, steps);
  // Update the route with calculated arc coordinates
  route.features[0].geometry.coordinates = arc;
  return route;
};

onMounted(() => {
  mapboxgl.accessToken = MAPBOX_TOKEN;
  const map = new mapboxgl.Map({
    container: 'map',
    style: STYLE_URL,
    zoom: 4,
    center: [37.6156, 55.7522]
  });

  // get map handle
  m = map;

  map.on('load', () => {
    // Here we want to load a layer
    map.addSource('ussr', {
      type: 'geojson',
      data:
      geoJSONSource.value,
    });
    map.addLayer({
      id: 'ussr-fill',
      type: 'fill',
      source: 'ussr',
      paint: {
        'fill-color': '#D8421C',
        'fill-opacity': 0.4
      },
    });

    // origins source
    map.addSource('origins', {
      type: 'geojson',
      data: {
        type: 'FeatureCollection',
        features: itemsFrom
      },
      cluster: true,
      clusterMaxZoom: 14,
      clusterRadius: 50
    });
    map.addLayer({
      id: 'origins-clusters',
      type: 'circle',
      source: 'origins',
      filter: ['has', 'point_count'],
      paint: {
        'circle-color': '#11b4da',
        'circle-radius': [
          'step',
          ['get', 'point_count'],
          20,
          100,
          30,
          750,
          40
        ]
      }
    });
    map.addLayer({
      id: 'origins-cluster-count',
      type: 'symbol',
      source: 'origins',
      filter: ['has', 'point_count'],
      layout: {
        'text-field': '{point_count_abbreviated}',
        'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
        'text-size': 12
      }
    });
    map.addLayer({
      id: 'unclustered-origins-points',
      type: 'circle',
      source: 'origins',
      filter: ['!', ['has', 'point_count']],
      paint: {
        'circle-color': '#11b4da',
        'circle-radius': 4,
        'circle-stroke-width': 1,
        'circle-stroke-color': '#fff'
      }
    });

    // destinations source
    map.addSource('destinations', {
      type: 'geojson',
      data: {
        type: 'FeatureCollection',
        features: itemsTo
      },
      cluster: true,
      clusterMaxZoom: 14,
      clusterRadius: 50
    });
    map.addLayer({
      id: 'destinations-clusters',
      type: 'circle',
      source: 'destinations',
      filter: ['has', 'point_count'],
      paint: {
        'circle-color': 'red',
        'circle-radius': [
          'step',
          ['get', 'point_count'],
          20,
          100,
          30,
          750,
          40
        ]
      }
    });

    map.addLayer({
      id: 'destinations-cluster-count',
      type: 'symbol',
      source: 'destinations',
      filter: ['has', 'point_count'],
      layout: {
        'text-field': '{point_count_abbreviated}',
        'text-font': ['DIN Offc Pro Medium', 'Arial Unicode MS Bold'],
        'text-size': 12
      }
    });

    map.addLayer({
      id: 'unclustered-destinations-points',
      type: 'circle',
      source: 'destinations',
      filter: ['!', ['has', 'point_count']],
      paint: {
        'circle-color': 'red',
        'circle-radius': 4,
        'circle-stroke-width': 1,
        'circle-stroke-color': '#fff'
      }
    });

  });
  // inspect a cluster on click
  map.on('click', 'origins-clusters', (e) => {
    const features = map.queryRenderedFeatures(e.point, {
      layers: ['origins-clusters']
    });
    const clusterId = features[0].properties.cluster_id;
    map.getSource('origins').getClusterExpansionZoom(
        clusterId,
        (err, zoom) => {
          if (err) return;
          map.easeTo({
            center: features[0].geometry.coordinates,
            zoom: zoom
          });
        }
    );
  });

  map.on('click', 'destinations-clusters', (e) => {
    const features = map.queryRenderedFeatures(e.point, {
      layers: ['destinations-clusters']
    });
    const clusterId = features[0].properties.cluster_id;
    map.getSource('destinations').getClusterExpansionZoom(
        clusterId,
        (err, zoom) => {
          if (err) return;
          map.easeTo({
            center: features[0].geometry.coordinates,
            zoom: zoom
          });
        }
    );
  });
  //
  // // show popup
  map.on('click', 'unclustered-origins-points', (e) => {
    // Copy coordinates array.
    console.log('clicked');
    const coordinates = e.features[0].geometry.coordinates;
    const destination = [e.features[0].properties.destination_lng, e.features[0].properties.destination_lat];
    console.log(`origin: ${coordinates}\n destination: ${destination}`);
    // const description = e.features[0].properties.description;
    const urlBase = e.features[0].properties.url_base;
    const urlImg = e.features[0].properties.url_front_img;
    const cityFrom = e.features[0].properties.city_from;
    const cityTo = e.features[0].properties.city_to;
    // const year = e.features[0].year;

    const postcardPopup = new mapboxgl.Popup()
        .setLngLat(coordinates)
        .setHTML(
            `<img src="${urlImg} width="450" height="350"">
            <p>${cityFrom} - ${cityTo}</p>
            <p><a href="${urlBase}">Подробнее</a></p>`)
        .addTo(map);

    const route = generateArcSource(coordinates, destination);


    if (map.getLayer('route')) {
      console.log('route needed to be removed');
      map.removeLayer('route');
      map.removeSource('route');

      map.addSource('route', {
        type: 'geojson',
        data: route
      });
      map.addLayer({
        id: 'route',
        source: 'route',
        type: 'line',
        paint: {
          'line-width': 2,
          'line-color': '#007cbf'
        }
      });
    } else {
      map.addSource('route', {
        type: 'geojson',
        data: route
      });
      map.addLayer({
        id: 'route',
        source: 'route',
        type: 'line',
        paint: {
          'line-width': 2,
          'line-color': '#007cbf'
        }
      });
    }
    postcardPopup.on('close', () => {
      map.removeLayer('route');
      map.removeSource('route');
    });
  });
  // map.on('click', 'exile-unclustered-points', (e) => {
  //   // Copy coordinates array.
  //   const coordinates = e.features[0].geometry.coordinates.slice();
  //   const destination = [e.features[0].properties.home_lon, e.features[0].properties.home_lat];
  //   const description = e.features[0].properties.description;
  //   const name = e.features[0].properties.name;
  //   const year = e.features[0].year;
  //
  //   new mapboxgl.Popup()
  //       .setLngLat(coordinates)
  //       .setHTML(description)
  //       .addTo(map);
  // });
});
</script>

<style>
#map {
  height: 95vh;
}

#slider {
  height: 5vh;
}
</style>
