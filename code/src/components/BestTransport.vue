<template>
    <div class="title">
        <b-navbar toggleable="lg" type="dark" variant="info">
            <b-navbar-brand class="ml-2">
                <b>{{ appName }}</b>
            </b-navbar-brand>
        </b-navbar>

        <div
            class="d-flex justify-content-center align-items-center"
            style="height: 100vh"
        >
            <b-form @submit="onSubmit">
                <b-form-group
                    id="destinationInput"
                    label="Destino"
                    label-for="destination"
                >
                    <b-form-select
                        v-model="destination"
                        :options="cityOptions"
                        placeholder="Selecione o destino"
                    ></b-form-select>
                </b-form-group>

                <b-form-group id="weightInput" label="Peso" label-for="weight">
                    <b-form-input
                        id="weight"
                        v-model="weight"
                        placeholder="Peso da carga em kg"
                    ></b-form-input>
                </b-form-group>

                <b-button type="submit" variant="primary">Analisar</b-button>
            </b-form>

            <div v-if="showResult">
                <b-card title="Mais rápido" class="mb-3">
                    <p class="card-text">Cost: {{ costFastest }}</p>
                </b-card>

                <b-card title="Mais barato" class="mb-3">
                    <p class="card-text">Cost: {{ costCheapest }}</p>
                </b-card>
            </div>
        </div>
    </div>
</template>

<script>
import { BNavbar, BNavbarBrand } from "bootstrap-vue";
import axios from "axios";

export default {
    components: {
        BNavbar,
        BNavbarBrand,
    },
    data() {
        const appName = "";

        return {
            appName,
            cityOptions: [],
            destination: "",
            weight: "",
            costFastest: 0,
            costCheapest: 0,
            showResult: false,
        };
    },
    created() {
        this.getData();
        this.appName = "Melhor Frete";
    },
    methods: {
        getData() {
            axios
                .get("http://localhost:3000/transport")
                .then((res) => {
                    const cities = [
                        ...new Set(res.data.map((item) => item.city)),
                    ];
                    this.cityOptions = cities;
                    this.data = res.data;
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        onSubmit(event) {
            event.preventDefault();
            console.log("destination:", this.destination);
            console.log("weight:", this.weight);

            let custoFinal = Infinity;
            let fastestLeadTime = Infinity;
            let fastestTransport = null;

            for (const transport of this.data) {
                if (this.destination === transport.city) {
                    const costLight = parseFloat(
                        transport.cost_transport_light.replace("R$ ", "")
                    );
                    const costHeavy = parseFloat(
                        transport.cost_transport_heavy.replace("R$ ", "")
                    );
                    const leadTime = parseFloat(
                        transport.lead_time.match(/\d+/)[0]
                    );

                    let currentCost = 0;

                    if (this.weight <= 100) {
                        currentCost = costLight;
                    } else {
                        currentCost = costHeavy;
                    }

                    if (currentCost < custoFinal) {
                        custoFinal = currentCost;
                    }

                    if (leadTime < fastestLeadTime) {
                        fastestLeadTime = leadTime;
                        fastestTransport = transport;
                    }
                }
            }

            this.costCheapest = custoFinal * parseFloat(this.weight);

            if (this.weight <= 100) {
                this.costFastest =
                    parseFloat(
                        fastestTransport.cost_transport_light.replace("R$ ", "")
                    ) * parseFloat(this.weight);
            } else {
                this.costFastest =
                    parseFloat(
                        fastestTransport.cost_transport_heavy.replace("R$ ", "")
                    ) * parseFloat(this.weight);
            }
            this.showResult = true;
        },
        methodFoo() {
            console.log(this.appName);
        },
    },
};
</script>

<style scoped>
.title .navbar {
    background-color: #00aca6 !important;
}

.title .navbar-brand {
    margin-left: 20px;
}
</style>
