<template>
    <div class="myContainer">
        <div v-if="mapError" class="mapError d-flex justify-content-center align-items-center">
            <p class="text-light fs-4">Falha ao encontrar o endereço no mapa...</p>
        </div>
        <div v-if="!mapError" id="map"></div>
    </div>
</template>

<script>
export default {
    name: 'Map',
    data() {
        return {
            mapReference: null,
            markerReference: null,
            geocoder: null,
            coordinates: { lat: 0, lng: 0 },
            addressMap: {},
            mapError: false
        }
    },
    props: {
        address: {
            type: String,
            required: true
        }
    },
    emits: ['addressMap'],
    created() {
        this.initMap();
    },
    watch: {
        async address() {
            await this.getCoordinates()
        }
    },
    methods: {
        //Initialize the map and emit an event with a data object
        async initMap() {
            const { Geocoder } = await google.maps.importLibrary('geocoding');
            const { Map } = await google.maps.importLibrary('maps');
            const { AdvancedMarkerElement } = await google.maps.importLibrary("marker");
            this.geocoder = new Geocoder();

            const map = new Map(document.getElementById('map'), {
                zoom: 17,
                center: this.coordinates,
                mapId: 'MyMapID456778'

            });

            this.mapReference = map

            await this.getCoordinates()

            const marker = new AdvancedMarkerElement({
                map,
                position: this.coordinates,
                gmpDraggable: true
            });

            this.markerReference = marker

            marker.addListener('dragend', () => {
                const position = marker.position;
                this.coordinates = { lat: position.lat, lng: position.lng };
                this.getAddress()
            });
        },

        //Search coordinates by address string
        async getCoordinates() {
            await this.geocoder.geocode({ address: this.address }, (results, status) => {
                if (status === 'OK' && results[0]) {
                    this.coordinates = {
                        lat: results[0].geometry.location.lat(),
                        lng: results[0].geometry.location.lng()
                    };

                    this.extractAddressInfo(results[0].address_components);

                    if (this.markerReference) this.markerReference.position = this.coordinates

                    this.mapReference.setCenter(this.coordinates);
                    return
                }

                this.mapError = true
            })
        },

        //Search the address by coordinates
        async getAddress() {
            await this.geocoder.geocode({ location: this.coordinates }, (results, status) => {
                if (status === 'OK' && results[0]) {
                    this.extractAddressInfo(results[0].address_components);
                    return
                }

                this.mapError = true
            });
        },

        //Organizes the address data returned by the Google Maps API and emits an event with a data object
        extractAddressInfo(addressComponents) {
            const mapping = {
                'postal_code': 'cep',
                'country': 'country',
                'locality': 'city',
                'administrative_area_level_2': 'city',
                'administrative_area_level_1': 'state',
                'route': 'publicPlace',
                'sublocality': 'neighborhood',
                'sublocality_level_1': 'neighborhood',
                'street_number': 'number',
                'subpremise': 'complement'
            };

            addressComponents.forEach(component => {
                component.types.forEach(type => {
                    if (mapping[type]) {
                        this.addressMap[mapping[type]] = component.long_name;
                    }
                });

                this.emitMapData()
            });
        },
        
        //Emits an event with map data in an object
        emitMapData() {
            this.$emit('addressMap', { addressMap: this.addressMap, coordinates: this.coordinates });
        }

    },
}
</script>

<style scoped>
#map {
    height: 100%;
    width: 100%;
}

.myContainer {
    height: 30rem;
    width: 70rem;
    position: relative;
}

.mapError {
    height: 100%;
    width: 100%;
    background-color: rgba(133, 131, 131, 0.61);
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 10;
}
</style>
