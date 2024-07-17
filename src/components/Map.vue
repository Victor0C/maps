<template>
    <div id="map"></div>
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
                addressMap: {}
	        }
	    },
	    props:{
	        address:{
	            type: String,
	            required: true
	        }
	    },
        emits:['addressMap'],
	    created() {
	        this.initMap();
	    },
        watch: {
           async address(newAddress) {
                await this.getCoordinates()
            }
        },
	    methods: {
	        async initMap(){
                const { Geocoder } = await google.maps.importLibrary('geocoding');
	            const { Map } = await google.maps.importLibrary('maps');
                const { AdvancedMarkerElement } = await google.maps.importLibrary("marker");
	            this.geocoder = new Geocoder();

	            const map = new Map(document.getElementById('map'), {
                    zoom: 17,
	                center: this.coordinates,
                    mapId:'MyMapID456778'
                    
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
	       async getCoordinates(){
	            await this.geocoder.geocode({ address: this.address }, (results, status)=>{
	                    if(status ==='OK' && results[0]){
                            this.coordinates = {
                                lat: results[0].geometry.location.lat(),
                                lng: results[0].geometry.location.lng()
                            };

                            if(this.markerReference) this.markerReference.position = this.coordinates

                            this.mapReference.setCenter(this.coordinates);
                            return
                        }

	                    console.log('Deu erro:' + status)
	            })
	        },
	        async getAddress(){
	            await this.geocoder.geocode({ location: this.coordinates }, (results, status) => {
	                if (status === 'OK' && results[0]) {
	                    const addressComponents = results[0].address_components;
	                    this.extractAddressInfo(addressComponents);
                       
	                } else {
	                    console.error('Erro geocodeLatLng: ' + status);
	                }
	            });
	        },
            extractAddressInfo(addressComponents) {
                addressComponents.forEach(component => {
                    if (component.types.includes('postal_code')) {
                    this.addressMap.cep = component.long_name;
                    }

                    if (component.types.includes('country')) {
                    this.addressMap.country = component.long_name;
                    }

                    if (component.types.includes('locality') || component.types.includes('administrative_area_level_2')) {
                    this.addressMap.city = component.long_name;
                    }

                    if (component.types.includes('administrative_area_level_1')) {
                    this.addressMap.state = component.long_name;
                    }

                    if (component.types.includes('route')) {
                    this.addressMap.publicPlace = component.long_name;
                    }

                    if (component.types.includes('sublocality') || component.types.includes('sublocality_level_1')) {
                    this.addressMap.neighborhood = component.long_name;
                    }

                    if (component.types.includes('street_number')) {
                    this.addressMap.number = component.long_name;
                    }

                    if (component.types.includes('subpremise')) {
                    this.addressMap.complement = component.long_name;
                    }
                });

                this.$emit('addressMap', this.addressMap);
            }
	    },
	}
</script>

<style scoped>
	#map {
		height: 400px;
		width: 100%;
	}
</style>
