<template>
    <div class="title">
        <b-navbar toggleable="lg" type="dark" variant="info">
            <b-navbar-brand class="ml-2">
            </b-navbar-brand>
        </b-navbar>

        <b-container class="main-container">
            <b-row>
                <b-navbar toggleable="lg" type="dark" variant="info">
                    <b-navbar-brand class="ml-2">
                        <b>{{ appName }}</b>
                    </b-navbar-brand>
                </b-navbar>
                <b-col class="form-col col col-4">
                    <h4>Insira o destino e o peso</h4>
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
                                class="form-input"
                            ></b-form-select>
                        </b-form-group>

                        <b-form-group
                            id="weightInput"
                            label="Peso"
                            label-for="weight"
                        >
                            <b-form-input
                                id="weight"
                                v-model="weight"
                                placeholder="Peso da carga em kg"
                                class="form-input"
                            ></b-form-input>
                        </b-form-group>

                        <div class="center-button">
                            <b-button
                                class="analyze-button"
                                type="submit"
                                variant="primary"
                                >Analisar</b-button
                            >
                        </div>

                        <b-modal
                            ref="modal"
                            hide-header
                            hide-footer
                            class="custom-modal"
                        >
                            <p>Insira os valores para realizar a análise.</p>
                            <b-button
                                variant="primary"
                                @click="$refs.modal.hide()"
                                >Fechar</b-button
                            >
                        </b-modal>
                    </b-form>
                </b-col>

                <b-col class="col col-8" v-if="showResult">
                    <h4>
                        Estas são as melhores alternativas de frete que
                        encontramos para você.
                    </h4>
                    <b-card
                        title="Frete com menor valor"
                        class="mb-3 cardContainer"
                    >
                        <p>Nome: {{ cheapestName }}</p>
                        <p>Preço: R$ {{ costCheapest }}</p>
                        <p>Tempo estimado: {{ cheapestTime }}</p>
                    </b-card>

                    <b-card title="Frete mais rápido" class="mb-3">
                        <p>Nome: {{ fastestName }}</p>
                        <p>Preço: R$ {{ costFastest }}</p>
                        <p>Tempo estimado: {{ fastestTime }}</p>
                    </b-card>

                    <b-button
                        @click="clearForm"
                        variant="primary"
                        class="clear-button"
                        >Limpar</b-button
                    >
                </b-col>
            </b-row>
        </b-container>
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
            fastestTime: 0,
            cheapestTime: 0,
            fastestName: "",
            cheapestName: "",
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

            if (!this.destination || !this.weight) {
                this.$refs.modal.show(); // Show the modal
                return;
            }

            let custoFinal = Infinity;
            let fastestLeadTime = Infinity;
            let fastestTransport = null;
            let cheapestTransport = null;

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
                        cheapestTransport = transport;
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
            this.fastestName = fastestTransport.name;
            this.cheapestName = cheapestTransport.name;

            this.fastestTime = fastestTransport.lead_time;
            this.cheapestTime = cheapestTransport.lead_time;
            this.showResult = true;
        },
        clearForm() {
            this.destination = "";
            this.weight = "";
            this.showResult = false;
            this.costCheapest = 0;
            this.costFastest = 0;
        },
        methodFoo() {
            console.log(this.appName);
        },
    },
};
</script>

<style scoped>
.title .navbar {
    background-color: #316982 !important;
}

.title .navbar-brand {
    margin-left: 20px;
}

.main-container {
    height: 640px;
    margin-top: 40px;
    box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
    border-radius: 10px;
}

.center-button {
    display: flex;
    justify-content: center;
}
.analyze-button {
    margin-top: 10px;
    background-color: #93b5c0;
    padding: 5px 40px;
    border: none;
    color: black;
}

.clear-button {
    margin-top: 10px;
    background-color: #93b5c0;
    padding: 5px 40px;
    border: none;
    color: black;
}

.custom-modal .modal-dialog {
    border-radius: 10px;
    max-width: 600px;
    width: 100%;
    margin: 1.75rem auto;
}

.form-input {
    width: 100%;
}

.form-col {
    margin-top: 20px;
    margin-bottom: 20px;
    height: 540px;
    background-color: #efeff2;
}
</style>
