<template>
    <div class="route">
        <h1> Picking Route </h1>
        <div>
            <div class="infoContainer" v-for="zone in zones" :key="zone.id">
                <div class="left">
                    <img class="hexagon" alt="hexagon" src="../assets/hex.svg" />
                    <p> {{zone.name}} </p>
                    <div class="line"></div>
                </div>
                <div class="products">
                    <v-data-table v-model="selected"
                        :headers="headers"
                        :items="zone.products"
                        item-key="idProduct"
                        hide-actions
                        class="elevation-0">
                        <template slot="items" slot-scope="props">
                            <td> <v-checkbox v-model="props.selected" primary hide-details></v-checkbox> </td>
                            <td class="text-xs-center">{{ props.item.sectionSuggestion }}</td>
                            <td class="text-xs-center">{{ props.item.artigo }}</td>
                            <td class="text-xs-center">{{ props.item.qnt}}</td>
                            <td> <v-text-field v-model=props.item.qntPicked></v-text-field>
                            <td class="text-xs-center">{{ props.item.orderNumDoc }}</td>
                            <td class="text-xs-center">{{ props.item.entity }}</td>
                        </template>
                    </v-data-table>
                </div>
            </div>
        </div>

        <div id="route-next-button">
        <!-- <v-btn to="/route" color="primary" round dark> Next </v-btn> -->
            <template>
                <v-layout justify-end>
                    <v-dialog v-model="dialog" hide-overlay dark max-width="290">
                        <v-btn slot="activator" color="primary" round dark>Next</v-btn>
                        <v-card>
                            <v-card-title class="headline">Are you sure you want to continue?</v-card-title>
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="primary" round dark @click="dialog = false">No</v-btn>
                                <v-btn color="primary" round dark @click="createPickingRequests(), dialog = false">Yes</v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-layout>
            </template>
        </div>
    </div>
</template>

<script>
    export default {
        data () {
            return {
                dialog: false,
                search: '',
                selected: [],
                productsList: [],
                headers: [
                    { text: 'Picked', sortable: false, align: 'left' },
                    { text: 'Section', sortable: false, align: 'center', value: 'section' },
                    { text: 'Product', sortable: false, align: 'center', value: 'artigo' },
                    { text: 'Qnt', sortable: false, align: 'center', value: 'qnt' },
                    { text: 'Qnt Picked', sortable: false, align: 'center', value: 'qntPicked' },
                    { text: 'Document ID', sortable: false, align: 'center', value: 'orderNumDoc' },
                    { text: 'Entity', sortable: false, align: 'center', value: 'entity' }
                ],
                zones: []
            }
        },
        created () {
            !this.$session.exists() ? this.$router.replace({ name: 'Login' }) : this.prepareRoute();
        },
        methods: {
            prepareRoute() {
               if(this.$session.has('routeProducts')){
                    let tempProd = this.$session.get('routeProducts');
                    this.$session.remove('routeProducts');

                    // let tempProd = [
                    //     { value: false, zone: 'A2', section: 'E', shelf: 1, product: 'Amet', order_id: 'RDA21DA1', client: 'Sit', qnt: 3 },
                    //     { value: false, zone: 'A2', section: 'G', shelf: 2, product: 'Consectetur', order_id: 'JRGE0YK4', client: 'Dolor', qnt: 5 },
                    //     { value: false, zone: 'A1', section: 'C', shelf: 2, product: 'Ipsum', order_id: 'JRGE0YK4', client: 'Dolor', qnt: 5 },
                    //     { value: false, zone: 'A1', section: 'D', shelf: 4, product: 'Sit', order_id: 'TSDFS123', client: 'Lorem', qnt: 1 },
                    //     { value: false, zone: 'A1', section: 'H', shelf: 1, product: 'Elit', order_id: 'RDA21DA1', client: 'Sit', qnt: 3 },
                    //     { value: false, zone: 'A1', section: 'J', shelf: 2, product: 'Sed', order_id: 'JRGE0YK4', client: 'Dolor', qnt: 5 },
                    //     { value: false, zone: 'A1', section: 'B', shelf: 1, product: 'Lorem', order_id: 'RDA21DA1', client: 'Sit', qnt: 3 }
                    // ];
                    tempProd.sort((a, b) => (a.sectionSuggestion > b.sectionSuggestion) ? 1 : ((b.sectionSuggestion > a.sectionSuggestion) ? -1 : 0));
                    console.log(tempProd);
                    this.createTables(tempProd);
               } else 
                    alert('No values');
            },

            createTables(products) {
                let [lastZone, lastId, j] = ['', -1, 0];
                for(let i = 0; i < products.length; i++){
                    products[i].idProduct = j;
                    products[i].qntPicked = products[i].qnt;
                    this.productsList.push(products[i]);

                    if(products[i].zoneSuggestion != lastZone){
                        
                        let zone = {
                            id: lastId + 1,
                            name: products[i].zoneSuggestion,
                            products: [
                                products[i]
                            ]
                        }
                        this.zones.push(zone);
                        lastZone = products[i].zoneSuggestion;
                        lastId++;
                    } else {
                        this.zones[lastId].products.push(products[i]);
                    }
                    j++;
                }
            },

            async createPickingRequests () {
                //TRANSFORMAÇAO DE DOCUMENTOS
                let type;
                let tempProd = this.productsList;
                let tempOrders=[];
                tempProd.sort((a, b) => (a.orderNumDoc > b.orderNumDoc) ? 1 : ((b.orderNumDoc > a.orderNumDoc) ? -1 : 0));

                let lastNumDoc = -1;
                for(let i = 0; i < tempProd.length; i++){
                    if(tempProd[i].orderNumDoc != lastNumDoc){

                        let tipoDocTemp = tempProd[i].orderTipoDoc == "ECL" ? "GR" : "VGR";
                        let tipoEntidadeTemp = tempProd[i].orderTipoDoc == "ECL" ? "C" : "F";
                        let type = tempProd[i].orderTipoDoc;
                        
                        let order = {
                            numDoc: tempProd[i].orderNumDoc,
                            tipoDoc: tempProd[i].orderTipoDoc,
                            tipoNewDoc: tipoDocTemp,
                            serie: tempProd[i].orderSerie,
                            entidade: tempProd[i].entity,
                            tipoEntidade: tipoEntidadeTemp
                        }

                        tempOrders.push(order);
                        lastNumDoc = tempProd[i].orderNumDoc;
                    }
                }

                for(let j = 0; j < tempOrders.length; j++){
                    await this.sendTransDocRequest(tempOrders[j]);
                }

                //TRANSFERÊNCIA DE ARMAZENS
                if (this.productsList[0].orderTipoDoc == "ECF")
                    await this.sendPickingWaveRequest();
                //await new Promise(resolve => setTimeout(resolve, 1000));     
                
                this.$router.replace('/home');
            },

            async sendPickingWaveRequest () {
                const [axios, docType] = [require('axios'), this.productsList[0].orderTipoDoc];
                let doc = { TipoDoc: 'TRA', Serie: 'A', Data: new Date().toLocaleDateString(), Moeda: 'EUR', LinhasOrigem: [] };

                this.productsList.forEach(s => {
                    doc.LinhasOrigem.push({ Artigo: s.artigo, Armazem: `${s.zone}.${s.section}`, Localizacao: `${s.zone}.${s.section}`, Lote: '', Quantidade: s.qnt, QPicked: s.qntPicked, INV_EstadoOrigem: 'DISP', ArmazemSugestao: `${s.zoneSuggestion}.${s.sectionSuggestion}`, LinhasDestino: [] });
                });

                /* if (docType === 'ECL') {
                    doc.LinhasOrigem.forEach(s => {
                        s.LinhasDestino.push({ Artigo: s.Artigo, Armazem: 'A5.Z', Localizacao: 'A5.Z', Lote: '', Quantidade: s.QPicked, INV_EstadoDestino: 'DISP' });
                        delete s.QPicked; delete s.ArmazemSugestao;
                    });
                }
                else if (docType === 'ECF') { */
                    doc.LinhasOrigem.forEach(s => {
                        s.LinhasDestino.push({ Artigo: s.Artigo, Armazem: s.ArmazemSugestao, Localizacao: s.ArmazemSugestao, Lote: '', Quantidade: s.QPicked, INV_EstadoDestino: 'DISP' });
                        delete s.QPicked; delete s.ArmazemSugestao;
                    });
                //}

                console.log('Sending picking wave request...');
                await axios({ 
                    method: 'post', url: 'http://localhost:2018/WebApi/Inventario/Transferencias/CreateTransfer',
                    headers: { 'Authorization': `Bearer ${this.$session.get('access')}`, 'Content-Type': 'application/json' },
                    data: doc
                })
                .then(response => console.log('Picking wave created!'))
                .catch(err => console.log(err));
            },

            async sendTransDocRequest (order) {
                console.log("Transforming " + order.tipoDoc + " to " + order.tipoNewDoc);
                const axios = require('axios');
                var d = new Date();
                console.log(order);
                let typeURL = order.tipoDoc == "ECL" ? "Vendas" : "Compras";
                let typeData = order.tipoDoc == "ECL" ? "DataVenc" : "DataIntroducao";
                let serie = order.tipoDoc == "ECL" ? "B" : "A";

                await axios({
                    method: 'post',
                    url: 'http://localhost:2018/WebApi/' + typeURL + '/Docs/TransformDocument/' + order.tipoDoc + '/' + order.serie + '/' + order.numDoc + '/000/true',
                    headers: { 
                        'Authorization': 'Bearer ' + this.$session.get('access'), 
                        'Content-Type': 'application/json',
                    },
                    data: {
                        "Tipodoc": order.tipoNewDoc,
                        "Serie": order.serie,
                        "Entidade": order.entidade,
                        "TipoEntidade": order.tipoEntidade,
                        "DataDoc": d.getDay() + "/" + d.getMonth() + "/" + d.getFullYear(),
                        typeData: d.getDay() + "/" + d.getMonth() + "/" + d.getFullYear()
                    },
                }).then((response) => {
                    console.log("Document Transformed");
                    //console.log(response.data.DataSet.Table);
                    return response;
                }).catch((error) => {
                    console.log("Error transforming");
                    console.log(error);
                    return null;
                });
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

    h1{
        font-weight: lighter;
        margin-bottom: 1em;
    }

    td >>>div.v-text-field__slot{
        border-bottom: 0.01em solid var(--baby) !important;
        width: 0.3em;
    }

    td >>>div.v-text-field__slot input{
        color: var(--baby) !important;
        text-align: center;
    }
   

    div.infoContainer {
        width: 100%;
        display: grid;
        grid-template-columns: 6em auto;
        grid-column-gap: 3em;
        min-height: 4em;
    }

    div.left {
        display: flex;
        flex-direction: column;
        position: relative;
        height: 100%;
        width: 6em;
    }

    div.left .hexagon {
        width: 100%;
    }

    div.left p {
        position: absolute;

        height: 2.7em;
        line-height: 2.7em;
        vertical-align: middle;

        width: 100%;
        margin: 0;
        font-weight: lighter;
        text-align: center;
        color: var(--gold);
        font-size: 2.5em;
    }

    div.line {
        width: 51.5%;
        border-right: 0.15em solid var(--gold);
        height: 100%;
    }

    div.infoContainer:last-child div.line {
        display: none;
    }

    div.products {
        margin-bottom:3em;
    }

    div.products p {
        line-height: 1.5em;
        padding-right: 1em;
        padding-left: 1em;
        margin-top: 0.5em;
        margin-bottom: 0.5em;
    }

</style>
