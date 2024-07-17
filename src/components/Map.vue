<template>
    <div v-if="mapError" class="mapError d-flex justify-content-center align-items-center">
        <p class="text-danger fs-3">Falha ao encontrar o endereço no mapa...</p>
    </div>
    <div v-if="!mapError" id="map"></div>
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

            //Inicializa o mapa
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

            //Busca as coordenadas pela string de endereço
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

	                    this.mapError = true
	            })
	        },

            //Busca o endereço pelas coordenadas
	        async getAddress(){
	            await this.geocoder.geocode({ location: this.coordinates }, (results, status) => {
	                if (status === 'OK' && results[0]) {
	                    const addressComponents = results[0].address_components;
	                    this.extractAddressInfo(addressComponents);
                       
	                } else {
                        this.mapError = true
	                }
	            });
	        },

            //Organiza os dados do endereço retornado pela API do Google Maps e emite um evento com um objeto dos dados
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
    .mapError{
        height: 400px;
		width: 100%;
        background-color: rgba(250, 30, 30, 0.062)
    }
</style>
