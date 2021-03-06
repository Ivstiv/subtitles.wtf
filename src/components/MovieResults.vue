<template>
    <div id="movieResults" v-if="movieResults.length > 0" class="movie-results">
        <h1 ref="title" class="title">Pick your movie</h1>
        <div class="grid-container">
            <main class="grid-item main">
                <div ref="slider" class="items" :class="{ active: isDown }"  
                    @mousedown="mouseDown" 
                    @mouseleave="mouseLeave" 
                    @mouseup="mouseUp" 
                    @mousemove="mouseMove" 
                    v-touch:swipe="stopAutoScrollAnimation">
                    <MovieCard 
                        v-for="(movie, id) in movieResults" 
                        v-bind:key="id" 
                        v-bind:title="movie.Title" 
                        v-bind:poster="movie.Poster" 
                        v-bind:imdbID="movie.imdbID"
                    ></MovieCard>
                </div>
            </main>
        </div>
    </div>
    
</template>

<script>
import {EventBus} from '../event-bus'
import MovieCard from './MovieCard.vue'

export default {

    components: {
        MovieCard
    },

    data() {
        return {
            slider: null,
            isDown: false,
            startX: null,
            scrollLeft: null,
            movieResults: [],
            scrollingAnimation: null,
        }
    },

    mounted() {
        // TitleSearch emits movie-results
        EventBus.$on('movie-results', movies => {
            this.movieResults = movies;
            if(this.$refs.slider)
                this.$refs.slider.scrollLeft = 0;
            clearInterval(this.scrollingAnimation);
            this.scrollingAnimation = setInterval(() => {
                this.$refs.slider.scrollLeft = this.$refs.slider.scrollLeft + 1;
            }, 20);
            this.$nextTick(() => {
                // The whole view is rendered, so I can safely access or query
                this.$scrollTo(this.$refs.title, {duration: 2000});
            })
        });

        // MovieCard emits selected-movie
        EventBus.$on('selected-movie', () => {
            document.getElementById('movieResults').classList.add('hidden');
            setTimeout(() => {
                this.movieResults = [];
            }, 1000);
        });
    },

    updated() {
        this.slider = this.$refs.slider;
    },

    methods:{
        mouseDown(e) {
            e.preventDefault();
            this.isDown = true;
            this.startX = e.pageX - this.slider.offsetLeft;
            this.scrollLeft = this.slider.scrollLeft;
            this.stopAutoScrollAnimation();
        },

        mouseMove(e) {
            if(!this.isDown) return;
            const x = e.pageX - this.slider.offsetLeft;
            const walk = x - this.startX;
            this.slider.scrollLeft = this.scrollLeft - walk;
        },

        mouseLeave() {
            this.isDown = false;
        },

        mouseUp() {
            this.isDown = false;
        },

        stopAutoScrollAnimation() {
            if(this.scrollingAnimation) {
                clearInterval(this.scrollingAnimation);
                this.scrollingAnimation = null;
            }
        }
    }
}
</script>

<style lang="scss" scoped>
@import '../styles/components/MovieResults.scss';
</style>