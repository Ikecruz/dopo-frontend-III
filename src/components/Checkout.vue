<template>
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 h-fit gap-10 w-full">
        <div class="flex flex-col gap-4" v-for="activity in cart" :key="activity._id">
            <div class="h-[180px] overflow-hidden w-full rounded-[5px]">
                <img :src="activity.image" class="w-full h-full object-cover" />
            </div>
            <div class="flex flex-col gap-[3px]">
                <p class="font-black page_title">{{ activity.title }}</p>
                <p class="">
                    <span class="page_title font-bold">Available Space</span>:
                    <span>{{ activity.spaces }}</span>
                </p>
                <p class="">
                    <span class="page_title font-bold">Booked Space</span>:
                    <span>{{ activity.bookedSpaces }}</span>
                </p>
                <p class="">
                    <span class="page_title font-bold">Location</span>:
                    <span>{{ activity.location }}</span>
                </p>
                <p class="">
                    <span class="page_title font-bold">Price</span>:
                    <span>£{{ activity.price }}</span>
                </p>
            </div>
            <button class="my_button bg-red-500" @click="removeFromCart(activity)">
                Remove from cart
            </button>
        </div>
    </div>
</template>

<script>
export default {
    props: ['cart'],
    methods: {
        removeFromCart: function(activity) {
            this.$emit('removeFromCart', activity)
        }
    }
}
</script>

<style scoped>
.my_button {
    background-color: red;
}
</style>