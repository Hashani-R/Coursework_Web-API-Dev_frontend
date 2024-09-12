<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col">
        <h1 style="background-color: #42b983" class="mb-1 py-3 text-white text-center">
          The Real-Time Train Tracking System - Sri Lanka
        </h1>
      </div>
    </div>
    <div class="row">
      <RealTimeMap />
    </div>
    <div class="row">
      <div class="col">
        <h5 style="background-color: #42b983" class="mt-1 py-3 text-white text-center">
          The Real-Time Train Tracking System
        </h5>
      </div>
    </div>
  </div>
</template>

<script setup>
import RealTimeMap from './components/RealTimeMap.vue'
</script>

<style>
@import 'bootstrap/dist/css/bootstrap.min.css';
</style>
