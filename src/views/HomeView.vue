<template>
	<div class="container mt-5 MyContainer d-flex">
		<form @submit.prevent="buildAddressString" class="forms me-5">
			<div class="mb-3">
				<label for="cep" class="form-label">CEP</label>
				<input
					type="text"
					class="form-control"
					id="cep"
					v-model="address.cep"/>
			</div>
			<div class="mb-3">
				<label for="logradouro" class="form-label">Logradouro</label>
				<input
					type="text"
					class="form-control"
					id="logradouro"
					v-model="address.publicPlace"/>
			</div>
			<div class="mb-3">
				<label for="bairro" class="form-label">Bairro</label>
				<input
					type="text"
					class="form-control"
					id="bairro"
					v-model="address.neighborhood"/>
			</div>
			<div class="mb-3">
				<label for="numero" class="form-label">N°</label>
				<input
					type="text"
					class="form-control"
					id="numero"
					v-model="address.number"/>
			</div>
			<div class="mb-3">
				<label for="complemento" class="form-label">Complemento</label>
				<input
					type="text"
					class="form-control"
					id="complemento"
					v-model="address.complement" />
			</div>
			<div class="mb-3">
				<label for="referencia" class="form-label">Ponto de Referência</label>
				<input
					type="text"
					class="form-control"
					id="referencia"
					v-model="address.reference" />
			</div>
			<button type="submit" class="btn btn-primary">Enviar</button>
		</form>
    <Map v-if="addressString != ''" :address="addressString" @addressMap="getMapData"></Map>		
	</div>
</template>

<script>
import Map from '../components/Map.vue'

	export default {
    name:'HomeView',
		data() {
			return {
				address: {
					cep: '',
					publicPlace: '',
					neighborhood: '',
					number: '',
					complement: '',
					reference: '',
				},
        addressString:'',
        addressMap:{}
			};
		},
    components:{
      Map
    },
		methods: {
      //Recupera os dados do endereço exibido no mapa
      getMapData(addressMap){
        this.addressMap = {...addressMap}
      },

      //Somente para simular a api de cep que é utilizada na empresa
      async getCepData(){
        try {
          const response = await fetch(`https://viacep.com.br/ws/${this.address.cep}/json/`);

          if (!response.ok) {
            throw new Error('Erro ao buscar o CEP');
          }

          const data = await response.json();

          this.address.city = data.localidade
          this.address.state = data.uf
        } 
        catch (error) {
          console.error('Error ao buscar o cep:', error);
        }
      }, 

      //Coloca o endereço numa string para buscar no mapa
      async buildAddressString(){
        await this.getCepData()

        this.addressString =
          `${this.address.number},
          ${this.address.publicPlace},
          ${this.address.neighborhood},
          ${this.address.city},
          ${this.address.state},
          Brasil`;
      }
		},
	};
</script>

<style scoped>
	.forms {
		width: 40rem;
	}
</style>
