<template>

    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="d-flex justify-content-center">
                    <h1 class="text-white aling-self-center">Bank Laravel Vue App</h1>
                </div>

                <div class="card">
                    <div class="card-header" style="background-color:#3E1360;">

                    <ul class="nav nav-tabs card-header-tabs" id="bologna-list" role="tablist">
                        <li class="nav-item">
                        <a class="nav-link active" href="#description" role="tab" aria-controls="description" aria-selected="true">Description</a>
                        </li>
                        <li class="nav-item">
                        <a class="nav-link"  href="#client" role="tab" aria-controls="client" aria-selected="false">Cadastro de Cliente</a>
                        </li>
                        <li class="nav-item">
                        <a class="nav-link" href="#detalhes" role="tab" aria-controls="detalhes" aria-selected="false">Detalhes</a>
                        </li>
                    </ul>

                    </div>

                    <div class="card-body">

                        <div class="tab-content mt-3">
                            <div class="tab-pane active" id="description" role="tabpanel">
                                <form @submit.prevent="moneyOption">
                                    <div class="form-row">
                                        <div class="form-group col-md-4">
                                            <label for="comboClient">Cliente</label>
                                            <select id="comboClient" class="form-control" v-model="formUpdate.id">
                                                <option value="" disabled>Selecione</option>
                                                <option v-for="client in clients" :key="client.id" v-bind:value="client.id" selected>
                                                    {{ client.name }}
                                                </option>
                                            </select>
                                        </div>
                                        <div class="form-group col-md-4">
                                            <label for="inputState">Operação</label>
                                            <select id="inputState" class="form-control" v-model="formUpdate.opt">
                                                <option value="0">Saldo</option>
                                                <option value="1">Depósito</option>
                                                <option value="2">Saque</option>
                                            </select>
                                        </div>
                                        <div class="form-group col-md-4">
                                            <label for="inputValue">Valor</label>
                                            <div id="app" class="empty">
                                            <money v-model="formUpdate.price"
                                                    v-bind="money"
                                                    class="form-input form-control" maxlength="15" id="opt-money" ></money>
                                            </div>
                                        </div>
                                    </div>

                                    <div class="alert alert-success" role="alert" id="successMessageOpt" style="display:none">
                                        Operação realizada com sucesso.
                                    </div>

                                    <div class="alert alert-danger" role="alert" style="display:none" id="erroMessageOpt">
                                    </div>

                                    <button type="submit" class="btn btn-success">Confirmar</button>
                                </form>
                            </div>

                            <div class="tab-pane" id="client" role="tabpanel" aria-labelledby="client-tab">
                                <form @submit.prevent="saveData">
                                    <div class="form-row">
                                        <div class="form-group col-md-4">
                                            <label for="inputClient">Cliente</label>
                                            <input v-model="form.name" type="text" class="form-control" id="inputClient">
                                        </div>
                                        <div class="form-group col-md-4">
                                            <label for="inputValue">Saldo Inicial</label>
                                            <div id="app" class="empty">
                                            <money v-model="form.price"
                                                    v-bind="money"
                                                    class="form-input form-control" maxlength="15"></money>
                                            </div>
                                        </div>
                                    </div>

                                    <div class="alert alert-success" role="alert" id="successMessage" style="display:none">
                                        Cliente cadastrado com sucesso veja em detalhes.
                                    </div>

                                    <div class="alert alert-danger" role="alert" v-if="form.errors.has('name')" v-text="form.errors.get('name')">
                                    </div>

                                    <button type="submit" class="btn btn-success">Salvar</button>
                                </form>
                            </div>

                            <div class="tab-pane" id="detalhes" role="tabpanel" aria-labelledby="detalhes-tab">
                                <div v-if="clients.length>0">
                                    <div v-for="client in clients" :key="client.id"  class="row">
                                        <div class="col-5">
                                            {{client.name}}
                                        </div>
                                        <div class="col-5">
                                            R$  {{formatPrice(client.money)}} #
                                        </div>
                                        <div class="col-2">
                                            <div class="d-flex justify-content-end">
                                                <span>
                                                    <font-awesome-icon icon="trash" v-on:click="deleteClient(client)" />
                                                </span>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div v-if="clients.length==0">
                                    <p>
                                        Nenhum cliente cadastrado.
                                    </p>
                                </div>
                            </div>
                        </div>


                    </div>
                </div>
            </div>
        </div>
    </div>

</template>

<script>
    import {VMoney} from 'v-money'

    export default {

    components: {VMoney},

        data(){
            return{
                clients:'',
                form: new Form({
                    name:'',
                    price: 0
                }),
                formUpdate: new Form({
                    id:'',
                    price: 0,
                    opt : 0
                }),
                money: {
                    decimal: ',',
                    thousands: '.',
                    prefix: 'R$ ',
                    suffix: ' #',
                    precision: 2,
                    masked: false /* doesn't work with directive */
                }
            }
        },
        // directives: {money: VMoney},
        methods:{
            getData(){
                axios.get('api/client').then((res)=>{
                    this.clients = res.data
                }).catch((error)=>{
                    if (error.response) {
                        this.form.errors.record(error.response.data.errors)
                    }
                })
            },
            saveData(){
                this.adjustNumber();
                let data =  new FormData();
                data.append('name', this.form.name)
                data.append('money', this.form.price)

                axios.post('api/client', data).then((res)=>{
                    this.form.reset();
                    this.getData();

                    $("#successMessage").show();
                    setTimeout(function(){
                         $("#successMessage").hide();
                    }, 3000);

                }).catch((error)=>{
                    this.form.errors.record(error.response.data.errors)
                })
            },
            deleteClient(c){
                let data = new FormData();
                data.append('_method', 'DELETE')

                axios.post('/api/client/'+c.id, data).then((res) =>{

                    this.clients = res.data
                }).catch((error) => {
                    this.form.errors.record(error.response.data.errors)
                })
            },
            getClientMoney(){

            },
            addZeroes(num) {
                return (num/1).toFixed(Math.max(((num+'').split(".")[1]||"").length, 2));
            },
            adjustNumber(){
                if(this.formUpdate.price<0){
                    this.formUpdate.price *= -1;
                }
                if(this.form.price<0){
                    this.form.price *= -1;
                }
            },
            moneyOption(){
                this.adjustNumber();
                if(this.formUpdate.id==""){
                    $("#erroMessageOpt").text("Favor fornecer os dados necessários!");
                    $("#erroMessageOpt").show();
                    setTimeout(function(){
                        $("#erroMessageOpt").hide();
                    }, 3000);
                }
                else{
                    axios.get('/api/client/'+this.formUpdate.id).then((res)=>{
                        let aux = res.data;
                        aux.money = this.addZeroes(aux.money);
                        if(this.formUpdate.opt > 0){

                            if(this.formUpdate.opt == 1){

                                aux.money = parseFloat(aux.money) + parseFloat(this.formUpdate.price);

                            }

                            if(this.formUpdate.opt == 2){

                                aux.money = parseFloat(aux.money) - parseFloat(this.formUpdate.price);

                            }
                            let data = new FormData();
                            data.append('_method', 'PATCH')
                            data.append('money', aux.money)
                            axios.post('/api/client/'+aux.id, data).then((res)=>{
                                    this.getData();
                                    $("#successMessageOpt").show();
                                    setTimeout(function(){
                                        $("#successMessageOpt").hide();
                                    }, 3000);
                                }).catch((error) => {
                                    $("#erroMessageOpt").text("Não foi possível realizar operação!");
                                    $("#erroMessageOpt").show();
                                    setTimeout(function(){
                                        $("#erroMessageOpt").hide();
                                    }, 3000);
                            })
                        }else{
                            this.formUpdate.price = aux.money;
                        }

                    }).catch((error)=>{
                        $("#erroMessageOpt").text("Não foi possível buscar o cliente!");
                        $("#erroMessageOpt").show();
                        setTimeout(function(){
                            $("#erroMessageOpt").hide();
                        }, 3000);
                    })
                }

            },

            formatPrice(value) {
                let val = (value/1).toFixed(2).replace('.', ',')
                return val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
            }
        },
        mounted() {
            this.getData()
            $('#bologna-list a').on('click', function (e) {
                e.preventDefault()
                $(this).tab('show');
            })
        }
    }

</script>
