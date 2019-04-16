<template>
	<div>
	  <ValidationObserver ref="obsMain">
        <v-card slot-scope="{ invalid, validated }">
		  <v-card-title>{{ formTitle }}</v-card-title>
		  <v-card-text>
		    <v-layout wrap>
			  <v-flex xs12 sm6 md6>
				<VTextFieldWithValidation rules="" data-vv-name="version" v-model="vmData.version" label="버전" readonly/>
			  </v-flex>
			  <v-flex xs12 sm6 md6>
				<VTextFieldWithValidation rules="" data-vv-name="createdDt" v-model="vmData.createdDt" label="생성일시" readonly/>
			  </v-flex>
			  <v-flex xs12 sm12>
				<VTextFieldWithValidation rules="max:100" data-vv-name="description" v-model="vmData.description" label="설명"/>
			  </v-flex>
			</v-layout>
			  <v-data-table
				:headers="headers"
				:items="vmData.vms"
				class="elevation-1"
				hide-actions
			  >
				<template v-slot:headers="props">
				  <tr>
					<th :class="headerClass" rowspan="2">Name</th>
					<th :class="headerClass" rowspan="2">Cores</th>
					<th :class="headerClass" rowspan="2">Memory</th>
					<th :class="headerClass" rowspan="2">NW<br>Speed</th>
					<th :class="headerClass" rowspan="2">Available<br>CPU</th>
					<th :class="headerClass" rowspan="2">Available<br>Mem</th>
					<th :class="headerClass" colspan="3">Shared List</th>
					<th :class="headerClass" colspan="3">Dedicated List</th>
					<th :class="headerClass" v-if="editable" rowspan="2">Actions</th>
				  </tr>
				  <tr>
					<th :class="headerClass">Price/Hour</th>
					<th :class="headerClass">Price/Month</th>
					<th :class="headerClass">SK Price/Month</th>
					<th :class="headerClass">Price/Hour</th>
					<th :class="headerClass">Price/Hour</th>
					<th :class="headerClass">SK Price/Month</th>
				  </tr>
				</template>
				<template v-slot:items="props">
				  <td class="text-xs-left">{{ props.item.name }}</td>
				  <td class="text-xs-right">{{ props.item.core }}</td>
				  <td class="text-xs-right">{{ props.item.memory }}</td>
				  <td class="text-xs-right">{{ props.item.nwSpeed }}</td>
				  <td class="text-xs-right">{{ props.item.core - iksGeneral.platformCpuPerWorker }}</td>
				  <td class="text-xs-right">{{ props.item.memory - iksGeneral.platformMemoryPerWorker }}</td>
				  <td class="text-xs-right">{{ props.item.sharedPricePerHour | formatNumber }}</td>
				  <td class="text-xs-right">{{ props.item.sharedPricePerHour | toMonthlyPrice | formatNumber }}</td>
				  <td class="text-xs-right">{{ props.item.sharedPricePerHour | toMonthlySKPrice(iksGeneral.ibmDcRate) | formatNumber }}</td>
				  <td class="text-xs-right">{{ props.item.dedicatedPricePerHour | formatNumber }}</td>
				  <td class="text-xs-right">{{ props.item.dedicatedPricePerHour | toMonthlyPrice | formatNumber }}</td>
				  <td class="text-xs-right">{{ props.item.dedicatedPricePerHour | toMonthlySKPrice(iksGeneral.ibmDcRate) | formatNumber }}</td>
				  <td class="justify-center layout px-0" v-if="editable">
					<v-icon
					  small
					  class="mr-2"
					  @click="editItem(props.item)"
					>
					  edit
					</v-icon>
					<v-icon
					  small
					  @click="deleteItem(props.item)"
					>
					  delete
					</v-icon>
				  </td>
				</template>
			  </v-data-table>
	      </v-card-text>
 	      <v-card-actions>
 	        <v-btn small left color="primary" v-if="editable" @click="openDialog">VM 추가</v-btn>
		    <v-spacer></v-spacer>
		    <v-btn right color="primary" v-if="editable" @click="save" v-bind:disabled="invalid">저장</v-btn>
		    <v-btn color="blue darken-1" v-if="!editable" flat @click="closeDetailDialog">닫기</v-btn>
	      </v-card-actions>
        </v-card>
      </ValidationObserver>
        
	  <v-dialog v-model="dialog" max-width="400px">
		  <ValidationObserver ref="obs">
		    <v-card slot-scope="{ invalid, validated }">
			  <v-card-title>{{ formDialogTitle }}</v-card-title>
			  <v-card-text>
			    <v-container grid-list-md>
				  <v-layout wrap>
				    <v-flex xs12>
					  <VTextFieldWithValidation rules="required|max:50" data-vv-name="name" v-model="editedItem.name" label="Name" />
				    </v-flex>
				    <v-flex xs12>
				  	  <VTextFieldWithValidation rules="required|numeric" data-vv-name="core" v-model="editedItem.core" label="Cores" />
				    </v-flex>
				    <v-flex xs12>
					  <VTextFieldWithValidation rules="required|numeric" data-vv-name="memory" v-model="editedItem.memory" label="Memory" />
				    </v-flex>
				    <v-flex xs12>
					  <VTextFieldWithValidation rules="required|numeric" data-vv-name="nwSpeed" v-model="editedItem.nwSpeed" label="NW Speed (Gbps)" />
				    </v-flex>
				    <v-flex xs12>
					  <VTextFieldWithValidation rules="required|numeric" data-vv-name="sharedPricePerHour" v-model="editedItem.sharedPricePerHour" label="Shared List Price(원)/Hour" />
				    </v-flex>
				    <v-flex xs12>
					  <VTextFieldWithValidation rules="required|numeric" data-vv-name="dedicatedPricePerHour" v-model="editedItem.dedicatedPricePerHour" label="Dedicated List Price(원)/Hour" />
				    </v-flex>
				  </v-layout>
			    </v-container>
			  </v-card-text>
			  <v-card-actions>
			    <v-spacer></v-spacer>
			    <v-btn color="blue darken-1" flat @click="closeDialog">취소</v-btn>
			    <v-btn color="blue darken-1" flat @click="saveDialog" v-bind:disabled="invalid">저장</v-btn>
			  </v-card-actions>
		    </v-card>
		  </ValidationObserver>
	  </v-dialog>
	</div>
</template>

<script>
export default {
	data: () => ({
		headers: [],
		headerClass: "grey lighten-1 body-2 text-weight-bold",
		dialog: false,
		editedIndex: -1,
		editedItem: {},
		defaultItem: {},
		
		iksGeneral: {},
		vmData: {}
	}),
	props: [
		'versionId',
		'editable'
	],
	computed: {
		formTitle() {
			return this.editable ? 'IKS VM 비용의 최신 버전을 조회 및 수정합니다.' : 'IKS VM History Detail';
		},
		formDialogTitle () {
			return this.editedIndex === -1 ? 'VM 추가' : 'VM 수정';
		}
    },
	watch: {
		versionId: function() {
			this.$http.get('/iks_costs/vm/history/' + this.versionId).then(response => {
				this.vmData = Object.assign({}, response.data);
			})
		},
		dialog (val) {
			val || this.closeDialog();
		}
    },
    filters: {
    	toMonthlyPrice: function(value) {
    		return value * 24 * 31;
    	},
    	toMonthlySKPrice: function(value, dcRate) {
    		return value * 24 * 31 * (1 - dcRate/100);
    	}
    },
	created () {
		this.initialize();
    },
	methods: {
		test() {
			return 12345;
		},
		initialize () {
			this.$http.get('/general').then(response => {
				this.iksGeneral = response.data;
			})
			this.$http.get('/iks_costs/vm').then(response => {
				this.vmData = response.data;
			})
		},
		editItem (item) {
			this.editedIndex = this.vmData.vms.indexOf(item);
			this.editedItem = Object.assign({}, item);
			this.dialog = true;
		},
		deleteItem (item) {
			const index = this.vmData.vms.indexOf(item);
			confirm('삭제하시겠습니까?') && this.vmData.vms.splice(index, 1);
		},
		openDialog() {
			this.editedIndex = -1;
			this.editedItem = Object.assign({}, this.defaultItem);
			this.dialog = true;
		},
		closeDialog () {
			this.dialog = false;
			this.$refs.obs.reset();
			setTimeout(() => {
				this.editedItem = Object.assign({}, this.defaultItem);
				this.editedIndex = -1;
			}, 300);
		},
		saveDialog () {
			this.$refs.obs.validate().then(valid => {
				if(valid) {
					if (this.editedIndex > -1) {
						Object.assign(this.vmData.vms[this.editedIndex], this.editedItem);
					} else {
						this.vmData.vms.push(this.editedItem);
					}
					this.closeDialog();
				}
			});
		},
		save () {
			this.$refs.obsMain.validate().then(valid => {
				if(valid) {
					if(confirm('변경된 내용을 저장하시겠습니까?')) {
						this.$http.put('/iks_costs/vm', this.vmData).then(response => {
							alert("저장되었습니다.");
							this.initialize();
							this.$refs.obsMain.reset();
							this.$emit('fire-saved');
						});
					}
				}
			});
		},
		closeDetailDialog() {
			this.$emit('fire-dialog-closed');
		}
	}
}
</script>
