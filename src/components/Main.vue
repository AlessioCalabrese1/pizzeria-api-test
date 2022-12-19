<template>
    <section>
        <div class="container">
            <button class="btn btn-primary" @click="createPizza()">Create</button>
            <form @submit.prevent="saveNewPizza(newPizza)" v-if="pizza_create_id === 1">
                <label for="name">Name</label>
                <input type="text" name="name" v-model="newPizza.name">
                <br>
                <label for="description">Description</label>
                <input type="text" name="description" v-model="newPizza.description">
                <br>
                <label for="price">Price</label>
                <input type="number" name="price" v-model="newPizza.price">
                <br><br>
                <input type="submit" value="CREATE">
            </form>
            <div class="row">
                <div class="col-3" v-for="pizza in pizze" :key="pizza.id">
                    <div class="card" style="width: 18rem;">
                        <div class="card-body">
                            <h5 class="card-title">{{ pizza.name }}</h5>
                            <p class="card-text">{{ pizza.description }}.</p>
                            <p class="card-text">{{ pizza.price }}.</p>
                            <button class="btn btn-primary" @click="changeDefaultPizzaId(pizza.id)">Edit</button>
                            <button class="btn btn-danger" @click="changeDefaultPizzaId(pizza.id), deletePizza(pizza.id)">Delete</button>

                            <form @submit.prevent="updatePizza(pizza)" v-if="pizza_edit_id === pizza.id">
                                <label for="name">Name</label>
                                <input type="text" name="name" v-model="pizza.name">
                                <br>
                                <label for="description">Description</label>
                                <input type="text" name="description" v-model="pizza.description">
                                <br>
                                <label for="price">Price</label>
                                <input type="number" name="price" v-model="pizza.price">
                                <br><br>
                                <input type="submit" value="UPDATE">
                            </form>

                            <button class="btn btn-primary" @click="getIngredients(pizza.id)">Get Ingredients</button>

                            <div v-if="check"></div>

                            <ul v-if="pizza.ingredients">
                                <li v-for="ingredient in pizza.ingredients" :key="ingredient.id">
                                    {{ ingredient.name }}
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<script>
import axios from "axios";

const API_URL = "http://localhost:8080/api/1/pizzeria";
const API_INGREDIENT_URL = "http://localhost:8080/api/1/ingredienti";
const PIZZA_ID_EDIT_DEFAULT = -1;

export default {
    name: "Main",
    data() {
        return {
            pizze: [],
            pizza_edit_id: PIZZA_ID_EDIT_DEFAULT,
            pizza_create_id: -1,
            newPizza: { },
            check: false
        }
    },

    methods: {
        getPizzaIndexById(id) {
            for (let x = 0; x < this.pizze.length; x++) {
                const pizza = this.pizze[x];
                if (pizza.id == id)
                    return x;
            }
            return -1;
        },

        getAllPizza() {
            axios.get(API_URL)
                .then(response => {
                    this.pizze = response.data;
                }).catch(err => {
                    console.log(err);
                })
        },

        changeDefaultPizzaId(id) {
            this.pizza_edit_id = id;
        },

        updatePizza(pizza) {
            axios.post(API_URL + "/update/" + this.pizza_edit_id, pizza)
                .then(response => {
                    this.pizze[this.getPizzaIndexById(pizza.id)] = response.data;
                })
        },

        deletePizza(id) {
            axios.get(API_URL + "/delete/" + id)
                .then(response => {
                    if (!response.data) {
                        return;
                    }
                    this.pizze.splice(this.getPizzaIndexById(id), 1);
                }).catch(error => {
                    console.error(error);
                })
        },

        createPizza(){
            this.pizza_create_id = 1;
        },

        saveNewPizza(){
            axios.post(API_URL + "/create", this.newPizza)
            .then(response => {
                this.pizze.push(this.newPizza);
            }).catch(error => {
                console.error(error);
            })
        },

        getIngredients(id){
            this.check = true;
            axios.get(API_INGREDIENT_URL + "/by/pizza/" + id)
            .then(response => {
                console.log(response.data);
                this.pizze[this.getPizzaIndexById(id)].ingredients = response.data;
                this.check = false;
            }).catch(error => {
                console.error(error);
            })

        }
    },

    mounted() {
        this.getAllPizza();
    }
}
</script>

<style>

</style>