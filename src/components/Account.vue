<template>
    <div>
        <div class="information" v-if="loaded">
            <h1>Información de productos</h1>
            <h2>Nombre: <span>{{ name }}</span></h2>
            <h2>Email:  <span>{{ email }}</span></h2>
            <h2>Saldo:  <span>{{ balance }}</span></h2>
        </div>
    </div>
</template>


<script>
    import axios      from 'axios';
    import jwt_decode from 'jwt-decode';

    export default{
        name: "Account",

        data: function(){
            return {
                loaded  : false,
                name    : "",
                email   : "",
                balance : 0
            }
        },

        methods:{
            getData: async function(){
                if(localStorage.getItem("tokenRefresh") === null || localStorage.getItem("tokenAccess") === null) {
                    this.$emit("logOut");
                    return;
                }

                await this.verifyToken();
                let token  = localStorage.getItem("tokenAccess");
                let userId = jwt_decode(token).user_id.toString();

                axios.get(
                    `http://localhost:8000/user/${userId}/`,
                    {headers: {'Authorization': `Bearer ${token}`}}
                )
                .then((result) => {
                    this.loaded  = true;
                    this.name    = result.data.name;
                    this.email   = result.data.email;
                    this.balance = result.data.account.balance;
                })
                .catch((error) => {
                    this.$emit("logOut");
                })
            },

            verifyToken: async function(){
                return axios.post(
                        'http://localhost:8000/refresh/',
                        {refresh: localStorage.getItem("tokenRefresh")},
                        {headers:{}}
                    )
                    .then((result) => {
                        localStorage.setItem("tokenAccess", result.data.access);
                    })
                    .catch((error) => {
                        this.$emit("logOut");
                    })
            }
        },

        created: async function(){
            this.getData();
        }
    }
</script>


<style>
    .information{
        margin: 0;
        padding: 0%;
        width: 100%;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    .information h1{
        font-size: 60px;
        color: #0f1316;
    }

    .information h2{
        font-size: 40px;
        color: #283747;
    }

    .information span{
        color: crimson;
        font-weight: bold;
    }
</style>