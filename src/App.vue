<template>
  <div id="app">
    <Map locale="vi">
      <BaseMapControl position="bottom-left" />
      <PrintControl />
      <GeolocateControl />
      <HomeControl />
      <ZoomControl />

      <FullScreenControl />
      <MouseCoordinatesControl />
    </Map>
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */
import {
  FullScreenControl,
  MouseCoordinatesControl,
  ZoomControl,
  HomeControl,
  GeolocateControl,
  BaseMapControl,
  Map,
  PrintControl
} from "@components/Map";
export default {
  name: "App",
  components: {
    FullScreenControl,
    MouseCoordinatesControl,
    ZoomControl,
    HomeControl,
    GeolocateControl,
    BaseMapControl,
    Map,
    PrintControl
  }
};
</script>
<style>
* {
  padding: 0;
  margin: 0;
}

body,
html,
#app {
  height: 100%;
}
</style>
