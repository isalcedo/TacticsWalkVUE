<template>
    <div id="app_container">
        <mapSimple/>
    </div>
</template>

<script>
    import mapSimple from './components/mapSimple.vue'

    export default {
        name:       'app',
        components: {
            mapSimple
        }
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
