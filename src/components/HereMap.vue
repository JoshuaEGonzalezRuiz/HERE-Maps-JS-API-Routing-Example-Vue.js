
<template>
  <div id="map">
  <!--In the following div the HERE Map will render-->
    <div id="mapContainer" style="height:600px;width:100%" ref="hereMap"></div>
  </div>
</template>

<script>
export default {
  name: "HereMap",
  props: {
    center: Object
  },
  data() {
    return {
      platform: null,
      apikey: "YOUR_API_KEY"
      // You can get the API KEY from developer.here.com
    };
  },
  async mounted() {
    // Initialize the platform object:
    const platform = new window.H.service.Platform({
      apikey: this.apikey
    });
    this.platform = platform;
    this.initializeHereMap();
  },
  methods: {
    initializeHereMap() { // rendering map

      const mapContainer = this.$refs.hereMap;
      const H = window.H;
      // Obtain the default map types from the platform object
      var maptypes = this.platform.createDefaultLayers();

      // Instantiate (and display) a map object:
      var map = new H.Map(mapContainer, maptypes.vector.normal.map, {
        zoom: 10,
        center: this.center
        // center object { lat: 40.730610, lng: -73.935242 }
      });

      addEventListener("resize", () => map.getViewPort().resize());

      // add behavior control
      new H.mapevents.Behavior(new H.mapevents.MapEvents(map));

      // add UI
      H.ui.UI.createDefault(map, maptypes);
      // End rendering the initial map

      this.doRouting(map, H)
    },
    doRouting(map, H) {
        // Create the parameters for the routing request:
        var routingParameters = {
            //'routingMode': 'fast',
            'transportMode': 'truck',
            // The start point of the route:
            'origin': '48.98108,2.4574009',
            // The end point of the route:
            'destination': '48.98108,2.4574009',
            'via': new H.service.Url.MultiValueQueryParameter(['48.890263,2.3936799', '48.8510269,2.424703']),
            'departureTime': '2022-11-25T07:04:24+01:00',
            // Include the route shape in the response
            'return': 'polyline,summary,travelSummary'
        };

        // Define a callback function to process the routing response:
        var onResult = function (result) {
            // ensure that at least one route was found
            if (result.routes.length) {
                result.routes[0].sections.forEach((section) => {
                    // Create a linestring to use as a point source for the route line
                    let linestring = H.geo.LineString.fromFlexiblePolyline(section.polyline);

                    // Create a polyline to display the route:
                    let routeLine = new H.map.Polyline(linestring, {
                        style: { strokeColor: 'blue', lineWidth: 3 }
                    });

                    // Create a marker for the start point:
                    //let startMarker = new H.map.Marker(section.departure.place.location);

                    // Create a marker for the end point:
                    //let endMarker = new H.map.Marker(section.arrival.place.location);

                    // Add the route polyline and the two markers to the map:
                    map.addObject(routeLine);

                    // Set the map's viewport to make the whole route visible:
                    map.getViewModel().setLookAtData({ bounds: routeLine.getBoundingBox() });
                });
            }
        };

        // Get an instance of the routing service version 8:
        var router = this.platform.getRoutingService(null, 8);

        // Call calculateRoute() with the routing parameters,
        // the callback and an error callback function (called if a
        // communication error occurs):
        router.calculateRoute(routingParameters, onResult,
            function (error) {
                alert(error.message);
            });
    }
  }
};
</script>

<style scoped>
#map {
  width: 60vw;
  min-width: 360px;
  text-align: center;
  margin: 5% auto;
  background-color: #ccc;
}
</style>