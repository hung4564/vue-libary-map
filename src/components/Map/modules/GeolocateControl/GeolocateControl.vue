<template>
  <ModuleContainer v-bind="bindModule">
    <template #btn>
      <MapControlButton
        :disabled="disable"
        :icon="iconGeolocate"
        :tooltip="tooltipGeolocate"
        @click="onClick"
      />
    </template>
    <slot />
  </ModuleContainer>
</template>

<script>
import ModuleMixin from "@components/Map/mixins/ModuleMixin";

import { getLocation } from "@utils";
import { Marker } from "mapbox-gl";
//import MapControlGroupButton from "@components/Map/control/MapControlGroupButton.vue";
import MapControlButton from "@components/Map/control/MapControlButton.vue";
export default {
  components: { MapControlButton },
  mixins: [ModuleMixin],
  data: () => ({
    p_dotElement: null,
    p_circleElement: null,
    p_userLocationDotMarker: null,
    bindedOnZoomLayer: null,
    center: [],
    disable: false,
    active: false,
    p_accuracy: 0
  }),
  computed: {
    iconGeolocate() {
      if (this.disable) return "mdi-crosshairs-off";
      return "mdi-crosshairs-gps";
    },
    tooltipGeolocate() {
      if (this.disable)
        return this.trans(
          "map.action.geolocate-control-location-not-available"
        );
      return this.trans("map.action.geolocate-control-find-my-location");
    }
  },
  methods: {
    async onClick() {
      this.disable = false;
      if (!this.active) {
        await this.onGetLocation();
        if (this.center.length > 1)
          this.map.flyTo({
            center: this.center,
            zoom: 14,
            duration: 0
          });
      } else {
        this.active = false;
        this.onDestroy();
      }
    },
    onInit() {
      this.bindedOnZoomLayer = this.onZoom.bind(this);
    },
    onDestroy() {
      this.map.off("zoom", this.bindedOnZoomLayer);
      if (this.p_userLocationDotMarker) {
        this.p_userLocationDotMarker.remove();
      }
      if (this._accuracyCircleMarker) {
        this._accuracyCircleMarker.remove();
      }
    },
    onGetLocation() {
      return getLocation()
        .then((location) => {
          if (!location) {
            return;
          }
          this.active = true;
          this.location = {
            lat: location.latitude,
            lng: location.longitude,
            accuracy: location.accuracy
          };
          this.p_accuracy = location.accuracy;
          this.center = [this.location.lng, this.location.lat];
          this.onAddUi();
          this.map.on("zoom", this.bindedOnZoomLayer);
          this.$emit("update:location", this.location);
        })
        .catch((e) => {
          this.disable = true;
          console.error(e);
        });
    },
    onAddUi() {
      this.p_dotElement = DOMcreate("div", "mapboxgl-user-location");

      this.p_dotElement.classList.add("mapboxgl-user-location");
      this.p_dotElement.appendChild(
        DOMcreate("div", "mapboxgl-user-location-dot")
      );
      this.p_dotElement.appendChild(
        DOMcreate("div", "mapboxgl-user-location-heading")
      );
      this.p_userLocationDotMarker = new Marker({
        element: this.p_dotElement,
        rotationAlignment: "map",
        pitchAlignment: "map"
      });

      this.p_circleElement = DOMcreate(
        "div",
        "mapboxgl-user-location-accuracy-circle"
      );
      this._accuracyCircleMarker = new Marker({
        element: this.p_circleElement,
        pitchAlignment: "map"
      });
      this.p_userLocationDotMarker.setLngLat(this.center).addTo(this.map);
      this._accuracyCircleMarker.setLngLat(this.center).addTo(this.map);
      this.updateCircleRadius();
    },
    updateCircleRadius() {
      const y = this.map._container.getBoundingClientRect().height / 2;
      const a = this.map.unproject([0, y]);
      const b = this.map.unproject([100, y]);
      const metersPerPixel = a.distanceTo(b) / 100;
      const circleDiameter = Math.ceil(
        (2.0 * this.p_accuracy) / metersPerPixel
      );
      this.p_circleElement.style.width = `${circleDiameter}px`;
      this.p_circleElement.style.height = `${circleDiameter}px`;
    },
    onZoom() {
      this.updateCircleRadius();
    }
  }
};
const DOMcreate = function (tagName, className, container) {
  const el = window.document.createElement(tagName);
  if (className !== undefined) el.className = className;
  if (container) container.appendChild(el);
  return el;
};
</script>
