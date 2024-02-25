<script>
import Checkout from './components/Checkout.vue';
import Lessons from './components/Lessons.vue';

export default {
    name: "App",
    components: {
        Lessons,
        Checkout,
    },
    data() {
        return {
            activities: [],
            sortByProperty: "title",
            sortOrder: "ascending",
            searchKeyword: "",
            cart: [],
            viewActivities: true,
            checkout: {
                fullname: "",
                mobile: ""
            }
        }
    },
    methods: {
        getActivities: async function () {
            try {

                const data = await fetch("https://dopo-api.eu-west-2.elasticbeanstalk.com/activities");
                this.activities = await data.json()

            } catch (error) {
                console.log(error)
            }
        },
        /* Sorts the `activities` array based on the `sortByProperty`(title, location, price, space) 
        and `sortOrder`(ascending, decending) data properties. */
        sortActivities: function (activities, order) {
            return activities.sort((a, b) => {
                let aRefined = a[this.sortByProperty], bRefined = b[this.sortByProperty];

                if (typeof a[this.sortByProperty] === "string") {
                    aRefined = a[this.sortByProperty].toLowerCase();
                    bRefined = b[this.sortByProperty].toLowerCase();
                }

                return order === "ascending" ? aRefined > bRefined : aRefined < bRefined;
            })
        },
        /* Responsible for filtering the `activities` based on the `searchKeyword`. */
        searchActivities: async function () {
            try {

                const response = await fetch(`https://dopo-api.eu-west-2.elasticbeanstalk.com/activities/search?q=${this.searchKeyword}`)
                this.activities = await response.json()

            } catch (error) {
                console.log(error)
            }
        },
        /* Responsible for adding an activity to the cart. */
        addToCart: function (activity) {
            let itemInCart = this.cart.find(item => item._id === activity._id);

            if (!itemInCart) {
                let newItemInCart = {
                    ...activity,
                    bookedSpaces: 0
                }

                this.cart.push(newItemInCart);
            }

            this.activities = this.activities.map(item => {
                if (item._id === activity._id) return { ...item, spaces: item.spaces - 1 }
                return item
            });

            this.cart = this.cart.map(item => {
                if (item._id === activity._id) return {
                    ...item,
                    spaces: item.spaces - 1,
                    bookedSpaces: item.bookedSpaces + 1
                }
                return item
            });

        },
        removeFromCart: function (activity) {
            const cartWithOutRemovedActivity = this.cart.filter(item => item._id !== activity._id);

            this.cart = this.cart.map(item => {
                if (item._id === activity._id) {
                    return {
                        ...item,
                        bookedSpaces: item.bookedSpaces - 1,
                        spaces: item.spaces + 1
                    }
                }
                return item;
            })

            this.activities = this.activities.map(item => {
                if (item._id === activity._id) {
                    return {
                        ...item,
                        spaces: item.spaces + 1
                    }
                }

                return item;
            })

            if (activity.bookedSpaces <= 1) this.cart = cartWithOutRemovedActivity;
            if (this.cart.length === 0) this.viewActivities = true;

        },
        changePage: function () {
            this.viewActivities = !this.viewActivities;
        },
        acceptJustNumbers: function (event) {
            if (!/^[0-9+]+$/.test(event.key)) event.preventDefault();
        },
        acceptJustLetters: function (event) {
            if (!/^[a-zA-Z]+$/.test(event.key)) event.preventDefault();
        },
        finish: async function () {
            const order = {
                name: this.checkout.fullname,
                phoneNumber: this.checkout.mobile,
                items: this.cart
            }

            try {

                await fetch("https://dopo-api.eu-west-2.elasticbeanstalk.com/orders", {
                    method: "POST",
                    body: JSON.stringify(order),
                    headers: {
                        "Content-type": "application/json; charset=UTF-8"
                    }
                })

                await this.updateStock()

                alert("Order completed")

            } catch (error) {
                console.log(error)
            }
        },
        updateStock: async function () {
            try {

                await fetch("https://dopo-api.eu-west-2.elasticbeanstalk.com/activities", {
                    method: "PUT",
                    body: JSON.stringify(this.cart),
                    headers: {
                        "Content-type": "application/json; charset=UTF-8"
                    }
                })

            } catch (error) {
                console.log(error)
            }
        }
    },
    computed: {
        filteredActivities: function () {
            return this.sortActivities(
                this.activities,
                this.sortOrder
            );
        },
        /* Checks if an activity can be added to cart. It returns a function that takes an `id` parameter. */
        disableAddToCart: function () {
            return (id) => {
                const item = this.activities.find(item => item._id === id)
                if (item.spaces > 0) return false;
                return true
            }
        },
        totalPrice: function () {
            return this.cart.reduce((previousValue, currentValue) => {
                return previousValue + (currentValue.price * currentValue.bookedSpaces)
            }, 0)
        },
        validateForm: function () {
            return (this.checkout.fullname.length < 1 || this.checkout.mobile.length < 5)
        },
        cartLength: function () {
            return this.cart.length;
        }
    },
    created() {
        this.getActivities();
        if ("serviceWorker" in navigator) {
            navigator.serviceWorker.register("service-worker.js");
        }
    }
}

</script>

<template>
    <div id="app">

        <!-- NAVBAR -->
        <nav class="flex items-center justify-between h-[70px] my_container">
            <p class="logo text-2xl">Dopo</p>
            <div class="relative">
                <div class="h-[18px] w-[18px] rounded-[100%] bg-black flex items-center justify-center cart_indicator">
                    <p class="text-xs text-white">{{ cartLength }}</p>
                </div>
                <button class="rounded h-8 w-8 flex items-center justify-center bg-[#f5f5f5]" :disabled="cart.length < 1"
                    @click="changePage">
                    <i class="bi bi-bag"></i>
                </button>
            </div>
        </nav>

        <!-- PAGE TITLE AND IMAGE -->
        <div class="flex flex-wrap border-y border-[#e0e0e0]">
            <div class="w-full sm:w-[25%] h-[100px]">
                <img src="./assets/images/pencils.jpg" class="w-full h-full object-cover" />
            </div>
            <div
                class="flex-1 w-full sm:w-auto py-7 px-5 sm:px-10 sm:px-14 flex items-center justify-center sm:justify-start">
                <p class="text-2xl page_title font-black" v-if="viewActivities">After School Activities</p>
                <p class="text-2xl page_title font-black" v-else>Checkout</p>
            </div>
        </div>

        <div class="flex flex-wrap" v-if="viewActivities">
            <div class="w-full sm:w-[25%] sm:min-h-[77vh] border-e py-7 px-5 border-[#e0e0e0] relative">
                <div class="flex flex-col gap-5 sticky top-5">
                    <div class="flex flex-col gap-[5px]">
                        <label for="search_input" class="text-sm font-semibold">Keyword</label>
                        <input type="text" id="search_input" class="my_input" placeholder="Search by title and location"
                            v-model="searchKeyword" v-on:input="searchActivities" />
                    </div>

                    <div class="flex flex-col gap-[5px]">
                        <label for="sort_input" class="text-sm font-semibold">Sort by</label>
                        <select type="text" id="sort_input" class="my_input" placeholder="Sort by" v-model="sortByProperty">
                            <option value="title">Title</option>
                            <option value="location">Location</option>
                            <option value="price">Price</option>
                            <option value="spaces">Spaces</option>
                        </select>
                    </div>

                    <div class="flex flex-col gap-[5px]">
                        <label for="order_input" class="text-sm font-semibold">Order</label>
                        <select type="text" id="order_input" class="my_input" placeholder="Order" v-model="sortOrder">
                            <option value="ascending">Ascending</option>
                            <option value="descending">Descending</option>
                        </select>
                    </div>
                </div>
            </div>

            <Lessons :activities="filteredActivities" @addToCart="addToCart" />
            
        </div>

        <div class="flex flex-wrap" v-else>
            <div class="w-full sm:w-[70%] border-e sm:min-h-[77vh] py-7 px-5 border-[#e0e0e0]">
                <Checkout :cart="cart" @removeFromCart="removeFromCart" />
            </div>
            <div class="flex-1 w-full sm:w-auto py-7 px-5 sm:px-10 sm:px-14 flex sm:min-h-[77vh]">
                <div class="flex flex-col gap-5 sticky top-5 w-full">
                    <div class="flex flex-col gap-[5px]">
                        <label for="search_input" class="text-sm font-semibold">Name</label>
                        <input type="text" id="name_input" class="my_input" placeholder="Enter fullname"
                            v-model="checkout.fullname" @keypress="acceptJustLetters($event)" />
                    </div>
                    <div class="flex flex-col gap-[5px]">
                        <label for="search_input" class="text-sm font-semibold">Phone</label>
                        <input type="number" inputmode="numeric" id="phone_input" class="my_input"
                            placeholder="Enter contact number" v-model="checkout.mobile"
                            @keypress="acceptJustNumbers($event)" />
                    </div>
                    <div class="flex justify-between">
                        <p>Total</p>
                        <div class="flex">
                            <p class="text-xs">£</p>
                            <p class="text-2xl font-black page_title">{{ totalPrice }}</p>
                        </div>
                    </div>
                    <div class="flex flex-col gap-[5px]">
                        <button class="my_button" :disabled="validateForm" @click="finish()">
                            Checkout
                        </button>
                    </div>
                </div>
            </div>
        </div> 
    </div>
</template>

<style scoped>

</style>
