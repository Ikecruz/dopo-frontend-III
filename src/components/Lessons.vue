<template>
    <div class="flex-1 w-full sm:w-auto py-7 px-5 sm:px-10 sm:px-14 flex sm:min-h-[77vh]">
        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 h-fit gap-10 w-full"
            v-if="activities.length > 0">

            <div class="flex flex-col gap-4" v-for="activity in activities" :key="activity._id">
                <div class="h-[180px] overflow-hidden w-full rounded-[5px]">
                    <img :src="activity.image" class="w-full h-full object-cover" />
                </div>
                <div class="flex flex-col gap-[3px]">
                    <p class="font-black page_title">{{ activity.title }}</p>
                    <p class="">
                        <span class="page_title font-bold">Space</span>:
                        <span>{{ activity.spaces }}</span>
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
                <button class="my_button" @click="addToCart(activity)">
                    <i class="bi bi-bag"></i>
                    Add to cart
                </button>
            </div>
        </div>

        <div class="w-full h-full flex flex-col justify-center items-center" v-else>
            <i class="bi bi-emoji-astonished text-[6em]"></i>
            <p class="text-2xl font-black page_title mb-2">Nothing Found</p>
            <p>Can't find any activity with "{{ searchKeyword }}" as Title or Location</p>
        </div>
    </div>
</template>

<script>
export default {
    props: ['activities'],
    methods: {
        disableAddToCart: function () {
            return (id) => {
                const item = this.$props.activities.find(item => item._id === id)
                if (item.spaces > 0) return false;
                return true
            }
        },
        addToCart: function (activity) {
            this.$emit('addToCart', activity)
        }
    }
}
</script>