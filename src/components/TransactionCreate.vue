<template>
    <div class="transaction">
        <div class="containerTransaction">
            <h2>Realizar transferencia electrónica:</h2>
            <form v-on:submit.prevent="processTransaction">
                <select v-model="transaction.transaction_data.origin_account">
                    <option disabled selected>Seleccione una cuenta de origen...</option>
                    <option v-for="account in myAccounts" :key="account.id" :value="account.id">{{ account.user.name }} - Cuenta {{ account.id }}</option>
                </select>
                <select v-model="transaction.transaction_data.destiny_account">
                    <option disabled selected>Seleccione una cuenta de destino...</option>
                    <option v-for="account in accounts" :key="account.id" :value="account.id">{{ account.user.name }} - Cuenta {{ account.id }}</option>
                </select>
                <input type="number" placeholder="monto a transferir" v-model="transaction.transaction_data.amount">
                <input type="text" placeholder="nota" v-model="transaction.transaction_data.note">
                <button type="submit">Transferir</button>
            </form>
        </div>
    </div>
</template>


<script>
    import axios      from 'axios';
    import jwt_decode from 'jwt-decode';

    export default{
        name: "TransactionCreate",

        data: function(){
            return {
                transaction : {
                    user_id : 0,
                    transaction_data: {
                        origin_account  : 0,
                        destiny_account : 0, 
                        amount          : 0, 
                        register_date   : (new Date()).toJSON().toString(), 
                        note            : ""
                    }
                },
                accounts: [],
                myAccounts: []
            }
        },

        methods:{
            processTransaction: async function(){
                if(localStorage.getItem("tokenRefresh") === null || localStorage.getItem("tokenAccess") === null) {
                    this.$emit("logOut");
                    return;
                }

                await this.verifyToken();
                let token  = localStorage.getItem("tokenAccess");
                let userId = jwt_decode(token).user_id.toString();

                this.transaction.user_id = userId;
                console.log(this.transaction)
                console.log(token);
                axios.post(
                    'http://localhost:8000/transaction/create/',
                    this.transaction,
                    {headers: {'Authorization': `Bearer ${token}`}}
                )
                .then((result) => {
                    console.log("Then");
                    this.$emit("completedTransaction");
                })
                .catch((error) => {
                    console.log("Error");
                    if(error.response.status == "401") {
                        alert("Usted no está autorizado para realizar esta operación.");
                    }
                    else if(error.response.status == "400"){
                        alert("La transacción no se pudo procesar correctamente.\nRevise todos los datos e intente de nuevo.");
                    }
                })
            },

            verifyToken: async function(){
                return axios.post(
                        'http://localhost:8000/refresh/',
                        {refresh: localStorage.getItem("tokenRefresh")},
                        {headers:{}}
                    )
                    .then((result) => {
                        console.log("New access token");
                        localStorage.setItem("tokenAccess", result.data.access);
                    })
                    .catch((error) => {
                        this.$emit("logOut");
                    })
            },

            getAccountsList: async function(){
                if(localStorage.getItem("tokenRefresh") === null || localStorage.getItem("tokenAccess") === null) {
                    this.$emit("logOut");
                    return;
                }

                await this.verifyToken();
                let token  = localStorage.getItem("tokenAccess");
                let userId = jwt_decode(token).user_id.toString();

                axios.get(
                    `http://localhost:8000/account/clientAccounts/${userId}/`,
                    {headers: {'Authorization': `Bearer ${token}`}}
                )
                .then((result) => {
                    this.accounts = result.data;
                })
                .catch((error) => {
                    if(error.response.status == "401") {
                        alert("Usted no está autorizado para realizar esta operación.");
                    }
                    else if(error.response.status == "500"){
                        alert("La plataforma está presentando problemas.\nIntente de nuevo más tarde.");
                    }
                })
            },

            getMyAccountsList: async function(){
                if(localStorage.getItem("tokenRefresh") === null || localStorage.getItem("tokenAccess") === null) {
                    this.$emit("logOut");
                    return;
                }

                await this.verifyToken();
                let token  = localStorage.getItem("tokenAccess");
                let userId = jwt_decode(token).user_id.toString();

                axios.get(
                    `http://localhost:8000/account/list/${userId}/`,
                    {headers: {'Authorization': `Bearer ${token}`}}
                )
                .then((result) => {
                    this.myAccounts = result.data;
                })
                .catch((error) => {
                    if(error.response.status == "401") {
                        alert("Usted no está autorizado para realizar esta operación.");
                    }
                    else if(error.response.status == "500"){
                        alert("La plataforma está presentando problemas.\nIntente de nuevo más tarde.");
                    }
                })
            },
        },

        created: async function(){
            this.getAccountsList();
            this.getMyAccountsList();
        }
    }
</script>


<style>
    .transaction{
        margin: 0;
        padding: 0%;
        height: 100%;
        width: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .containerTransaction {
        border: 3px solid #283747;
        border-radius: 10px;
        width: 40%;
        height: 60%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .transaction h2{
        color: #283747;
    }

    .transaction form{
        width: 70%;
    }

    .transaction input{
        height: 40px;
        width: 100%;
        box-sizing: border-box;
        padding: 10px 20px;
        margin: 5px 0;
        border: 1px solid #283747;
    }

    .transaction select{
        height: 40px;
        width: 100%;
        box-sizing: border-box;
        padding: 10px 20px;
        margin: 5px 0;
        border: 1px solid #283747;
    }

    .transaction button{
        width: 100%;
        height: 40px;
        color: hsl(210, 8%, 91%);
        background: #283747;
        border: 1px solid #E5E7E9;
        border-radius: 5px;
        padding: 10px 25px;
        margin: 5px 0 25px 0;
    }

    .transaction button:hover{
        color: #E5E7E9;
        background: crimson;
        border: 1px solid #283747;
    }
</style>