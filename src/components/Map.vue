<template>
    <div id="map"></div>
</template>

<script>
	export default {
	    name: 'Map',
	    data() {
	        return {
	           geocoder: null,
	           coordinates: { lat: 0, lng: 0 },
               addressMap: {
                    cep: '',
                    logradouro: '',
                    bairro: '',
                    numero: '',
                    complemento: ''
	            }
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
	        this.initMap()
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
                
                await this.getCoordinates(map)
               
                const marker = new AdvancedMarkerElement({
                    map,
                    position: this.coordinates,
                    gmpDraggable: true
                });
                
                marker.addListener('dragend', () => {
                    const position = marker.position;
                    this.coordinates = { lat: position.lat, lng: position.lng };
                    this.getAddress()
                });
                
	        },
	       async getCoordinates(map){
	            await this.geocoder.geocode({ address: this.address }, (results, status)=>{
	                    if(status ==='OK' && results[0]){
                            this.coordinates = {
                                lat: results[0].geometry.location.lat(),
                                lng: results[0].geometry.location.lng()
                            };
                            
                            map.setCenter(this.coordinates);
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
                    
                    if (component.types.includes('route')) {
	                    this.addressMap.logradouro = component.long_name;
	                }
                    
                    if (component.types.includes('sublocality')) {
	                    this.addressMap.bairro = component.long_name;
	                } 
                    
                    if (component.types.includes('street_number')) {
	                    this.addressMap.numero = component.long_name;
	                }
	            });

                console.log({addressMap: this.addressMap, coordinatesMap: this.coordinates})
                // this.$emit('addressMap', this.addressMap)
	        },
	    },
	}
</script>

<style scoped>
	#map {
		height: 400px;
		width: 100%;
	}
</style>
