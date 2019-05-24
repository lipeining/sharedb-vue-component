<template>
    <div id="app">
        <img alt="Vue logo" src="./assets/logo.png">
        <!-- <HelloWorld msg="Welcome to Your Vue.js App" /> -->
        <Doc collectionName="examples" documentId="richtext" socketUrl="ws://localhost:8000" />
    </div>
</template>

<script>
    // import HelloWorld from './components/HelloWorld.vue'
    import Doc from './components/doc.vue'

    export default {
        name: 'app',
        components: {
            // HelloWorld,
            Doc
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