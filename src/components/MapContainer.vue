<template>
    <div>
        <form @submit="onSubmit" class="add-form">
            <div class="form-control">
                <label>District</label>
                <input type="text" v-model="district" name="district" placeholder="Enter Your district" />
            </div>
            <div class="form-control">
                <label>State</label>
                <input type="text" v-model="state" name="state" placeholder="Enter Your State" />
            </div>
            <div class="form-control">
                <label>Country</label>
                <input type="text" v-model="country" name="country" placeholder="Enter Your country" />
            </div>
            <div class="form-control">
                <label>Pincode</label>
                <input type="text" v-model="postalcode" name="postalcode" placeholder="Enter Your postalcode" />
            </div>
            <input type="submit" value="Save Information" class="btn btn-block" />
        </form>
    </div>
    <div ref="map-root" style="width: 600px;height:600px"></div>
</template>
<script>
import View from 'ol/View'
import Map from 'ol/Map'
import TileLayer from 'ol/layer/Tile'
import OSM from 'ol/source/OSM'
import VectorLayer from 'ol/layer/Vector'
import VectorSource from 'ol/source/Vector'
import { fromLonLat } from "ol/proj";
// import GeoJSON from 'ol/format/GeoJSON'
import 'ol/ol.css'

let source = new VectorSource();
let vectorLayer = new VectorLayer({
    source: source,
});

let rasterLayer = new TileLayer({
    source: new OSM(),
});

export default {
    name: 'MapContainer',
    mounted() {
        // this is where we create the OpenLayers map
        new Map({
            // the map will be created using the 'map-root' ref
            target: this.$refs['map-root'],
            layers: [
                // adding a background tiled layer
                new TileLayer({
                    source: new OSM() // tiles are served by OpenStreetMap
                }),
            ],

            // the map view will initially show the whole world
            view: new View({
                zoom: 0,
                center: [0, 0],
                constrainResolution: true
            }),
        });
    },
    methods: {
        onSubmit(e) {
            e.preventDefault();
            const position = {
                district: this.district,
                state: this.state,
                country: this.country,
                postalcode: this.postalcode
            }
            console.log(position);

            const getLocation = async () => {
                const res = await fetch(`https://nominatim.openstreetmap.org/search?county=${position.district}&state=${position.state}&country=${position.country}&format=geojson`);
                const data = await res.json();
                // this.data = data;
                console.log(data.features[0]);
                // console.log(data.features[0].geometry.coordinates);
                const longitude = data.features[0].geometry.coordinates[0];
                const latitude = data.features[0].geometry.coordinates[1];
                console.log(longitude, latitude);

                new Map({
                    target: this.$refs['map-root'],
                    layers: [rasterLayer, vectorLayer],
                    view: new View({
                        zoom: 0,
                        center: fromLonLat([longitude, latitude]),
                        constrainResolution: true,
                    }),
                });
            }
            getLocation()

        },


    }
}

</script>
