<template>
	<div class="container mt-5 MyContainer d-flex">
		<form @submit.prevent="submitForm" class="forms me-5">
			<div class="mb-3">
				<label for="cep" class="form-label">CEP</label>
				<input
					type="text"
					class="form-control"
					id="cep"
					v-model="address.cep"
					required />
			</div>
			<div class="mb-3">
				<label for="logradouro" class="form-label">Cidade</label>
				<input
					type="text"
					class="form-control"
					id="logradouro"
					v-model="address.city"
					required />
			</div>
			<div class="mb-3">
				<label for="logradouro" class="form-label">Estado</label>
				<input
					type="text"
					class="form-control"
					id="logradouro"
					v-model="address.state"
					required />
			</div>
			<div class="mb-3">
				<label for="logradouro" class="form-label">Logradouro</label>
				<input
					type="text"
					class="form-control"
					id="logradouro"
					v-model="address.logradouro"
					required />
			</div>
			<div class="mb-3">
				<label for="bairro" class="form-label">Bairro</label>
				<input
					type="text"
					class="form-control"
					id="bairro"
					v-model="address.bairro"
					required />
			</div>
			<div class="mb-3">
				<label for="numero" class="form-label">N°</label>
				<input
					type="text"
					class="form-control"
					id="numero"
					v-model="address.numero"
					required />
			</div>
			<div class="mb-3">
				<label for="complemento" class="form-label">Complemento</label>
				<input
					type="text"
					class="form-control"
					id="complemento"
					v-model="address.complemento" />
			</div>
			<div class="mb-3">
				<label for="referencia" class="form-label">Ponto de Referência</label>
				<input
					type="text"
					class="form-control"
					id="referencia"
					v-model="address.referencia" />
			</div>
			<button type="submit" class="btn btn-primary">Enviar</button>
		</form>
    <Map address="169, Rua 31 de Março, Centro, Retirolândia, Bahia" ></Map>		
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
          city:'',
          state:'',
					publicPlace: '',
					neighborhood: '',
					number: '',
					complement: '',
					reference: '',
				},
        addressString:''
			};
		},
    components:{
      Map
    },
		methods: {
			submitForm() {
				console.log(this.address);
			},
		},
	};
</script>

<style scoped>
	.forms {
		width: 40rem;
	}
</style>
