<template>
    <div class="loading">
        <ul class="flex-h">
            <li class="load1"></li>
            <li class="load2"></li>
            <li class="load3"></li>
        </ul>
    </div>
</template>

<script>
    export default {
        
    }
</script>

<style lang="scss">
    @mixin loading ($n:1){
        margin-top: 45px*$n;
        ul {
            width: 30px*$n;
            height: 24px*$n;
        }
    }
    .loading {
        width: 100%;
        height: 100%;
        background: #fff;
        position: absolute;
        left: 0;
        top: 0;
        z-index: 50;
        @include loading

        ul {
            position: absolute;
            top: 20%;
            left: 0;
            right: 0;
            margin: auto;
            justify-content: space-between;
            align-items: flex-end;

            li {
                width: 24%;
                background: #fff;
            }
            li.load1 {
                background: red;
                animation: loadplay .4s linear infinite alternate;
            }
            li.load2 {
                background: blue;
                animation: loadplay .4s linear .15s infinite alternate;
            }
            li.load3 {
                background: green;
                animation: loadplay .4s linear .3s infinite alternate;
            }
        }
    }
    [data-dpr="2"] .loading {
        @include loading(2);
    }
    [data-dpr="3"] .loading {
        @include loading(3);
    }
    @keyframes loadplay {
        from {
            height: 20%;
        }
        to {
            height: 100%;
        }
    }
</style>