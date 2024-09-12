<template>
  <div class="col-9">
    <div id="map" class="map"></div>
  </div>
  <div class="col-3">
    <div class="row">
      <div class="col-12">
        <h5 class="mt-2">Select Train Name :</h5>
        <select class="form-select" aria-label="select example" v-model="selectedTrainId">
          <option v-for="train in trains" :key="train.train_id" :value="train.train_id">
            {{ train.train_name }}
          </option>
        </select>
      </div>
    </div>
    <hr style="border-width: 3px;">
    <div class="row">
      <div class="col-12">
        <h5 class="mt-2">Select Station :</h5>
        <select class="form-select" aria-label="select example" 
        v-model="selectedStationId" @change="fetchTrainsList">
          <option v-for="station in stations" 
          :key="station.train_station_id" :value="station.station_id">
            {{ station.station_name }}
          </option>
        </select>
        <!-- <div class="d-grid gap-2 mt-2">
          <button style="background-color: rgb(66, 185, 131); border-color: rgb(66, 185, 131);" class="btn btn-primary" type="button">Show Trains</button>
        </div> -->
      </div>
    </div>
    <div class="row" v-if="trains2.length">
      <h5 class="mt-2">Search and Show Trains:</h5>
      <div style="overflow-y: scroll; height: 220px;">
        <div class="col-12" v-for="train in trains2" :key="train.train_id">
          <div class="d-grid mt-2">
            <button type="button" class="btn btn-outline-success btn-block" 
            @click="selectTrain(train.train_id)">
              {{ train.train_name }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="row" v-else>
      <h5 class="mt-2">Search and Show Trains:</h5>
      <div style="overflow-y: scroll; height: 220px;" class="d-flex align-items-center justify-content-center">
        <h6>Please select station to show trains!</h6>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <a href="https://seatreservation.railway.gov.lk/mtktwebslr/" target="_blank" role="button" 
        style="width: 100%; background-color: #42b983; border-color: #42b983;" 
        class="btn btn-primary mt-1">Booking Trains</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import L from 'leaflet';

const map = ref(null);
const marker = ref(null);

const trains = ref([]);
const trains2 = ref([]);
const selectedTrainId = ref(null);

const stations = ref([]);
const selectedStationId = ref(null);

// Fetch trains based on selected station
const fetchTrainsList = async () => {
  try {
    if (selectedStationId.value) {
      // Fetch routes associated with the selected station
      const routeResponse = await fetch('https://api.livetrainlocation.xyz/api/train-routes');
      const routes = await routeResponse.json();
      const filteredRoutes = routes.filter(route => route.start_station_id === selectedStationId.value);

      if (filteredRoutes.length > 0) {
        const routeIds = filteredRoutes.map(route => route.route_id);
        
        // Fetch schedules for the selected routes
        const scheduleResponse = await fetch('https://api.livetrainlocation.xyz/api/schedules');
        const schedules = await scheduleResponse.json();
        const filteredSchedules = schedules.filter(schedule => routeIds.includes(schedule.route_id));

        // Fetch all trains to get train names
        const trainResponse = await fetch('https://api.livetrainlocation.xyz/api/trains');
        const trainsData = await trainResponse.json();

        // Get train details from filtered schedules, including train_name
        const trainDetails = filteredSchedules.map(schedule => {
          const train2 = trainsData.find(tr => tr.train_id === schedule.train_id);
          return {
            train_id: schedule.train_id,
            train_name: train2 ? train2.train_name : `Train ${schedule.train_id}`, // Fallback if train_name is not found
          };
        });

        // Remove duplicates based on train_id
        const uniqueTrains = trainDetails.reduce((acc, current) => {
          const isDuplicate = acc.some(train => train.train_id === current.train_id);
          if (!isDuplicate) {
            acc.push(current);
          }
          return acc;
        }, []);

        trains2.value = uniqueTrains;
      } else {
        trains2.value = [];
      }
    }
  } catch (error) {
    console.error('Error fetching trains:', error);
  }
};

// Handle train selection
const selectTrain = (trainId) => {
  fetchTrainLocation2(trainId);
  selectedTrainId.value = trainId;
};

// Function to fetch real-time train location
async function fetchTrainLocation2(train_id) {
  try {
    if (train_id == null) {
      //code
    } else {
      const response = await fetch('https://api.livetrainlocation.xyz/api/real-location/'+train_id);
      const data = await response.json();
      if (data && data.longitude && data.latitude) {
        const { longitude, latitude } = data;
        if (!map.value) {
          initializeMap(latitude, longitude);
        } else {
          updateMarker(latitude, longitude);
        }
      }
    }
    
  } catch (error) {
    console.error('Error fetching train location:', error);
  }
}

const fetchTrains = async () => {
  try {
    const response = await fetch('https://api.livetrainlocation.xyz/api/trains');
    const data = await response.json();
    trains.value = data;
  } catch (error) {
    console.error('Error fetching train data:', error);
  }
};

const fetchStations = async () => {
  try {
    const response = await fetch('https://api.livetrainlocation.xyz/api/stations');
    const data = await response.json();
    stations.value = data;
  } catch (error) {
    console.error('Error fetching station data:', error);
  }
};

// Function to initialize the map
const initializeMap = (latitude, longitude) => {
  map.value = L.map('map', { resizeObserver: true }).setView([latitude, longitude], 13);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19
  }).addTo(map.value);

  marker.value = L.marker([latitude, longitude]).addTo(map.value);
}

// Function to update the marker's position
const updateMarker = (latitude, longitude) => {
  marker.value.setLatLng([latitude, longitude]);
  map.value.setView([latitude, longitude], map.value.getZoom());
}

// Function to fetch real-time train location
const fetchTrainLocation = async () => {
  try {
    
    if (selectedTrainId.value == null) {
      //code
    } else {
      const response = await fetch('https://api.livetrainlocation.xyz/api/real-location/'+selectedTrainId.value);
      const data = await response.json();
      if (data && data.longitude && data.latitude) {
        const { longitude, latitude } = data;
        if (!map.value) {
          initializeMap(latitude, longitude);
        } else {
          updateMarker(latitude, longitude);
        }
      }
    }
  } catch (error) {
    console.error('Error fetching train location:', error);
  }
}

// Resize handler to update map size
const resizeHandler = () => {
  if (map.value) {
    map.value.invalidateSize();
  }
}



onMounted(() => {
  fetchTrainLocation();
  fetchTrains();
  fetchStations();
  setInterval(fetchTrainLocation, 2000); // Update every 2 seconds
  window.addEventListener('resize', resizeHandler);
})

onUnmounted(() => {
  window.removeEventListener('resize', resizeHandler);
})
</script>

<style scoped>
#map {
  width: 100%;
  height: 500px;
  min-height: 300px;
}

.fixed {
    position: absolute;
    overflow-y: scroll;
}
</style>
