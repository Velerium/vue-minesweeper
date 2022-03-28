<template>
    <div @click="$emit('mineCheck', number)" class="tile"></div>
</template>

<script>
export default {
    
    data() {
        return {
            status: 'clear',
        }
    },

    props: {
        number: Number,
    }
}
</script>

<style lang="scss" scoped>

.tile {
    width: 25px;
    height: 25px;
    border: 1px solid black;

    &:hover {
        background-color: #ddd;
    }
}

</style>