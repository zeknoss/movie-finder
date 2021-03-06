<template>
    <section id="movie-details" :class="{'bg-gradient-dark': details === null}">
        <template v-if="details !== null">
            <div class="detail-container">
                <b-container>
                    <b-row>
                        <b-col cols="12" md="6">
                            <img :src="poster" :alt="details.Title" class="img-fluid">
                        </b-col>
                        <b-col cols="12" md="6" class="text-white">
                            <h1 class="mb-0">{{ details.Title }} <small>{{ details.Genre }}</small></h1>
                            <b-badge variant="primary" class="mb-3 p-2">{{ details.Released }}</b-badge>
                            <b-badge variant="primary" class="mb-3 p-2">{{ details.imdbID }}</b-badge>
                            <h4><i class="fad fa-star text-info" /> {{ details.imdbRating }} <small>({{ details.imdbVotes }} Votes)</small></h4>
                            <dl>
                                <dt>Actors</dt>
                                <dd>{{ details.Actors }}</dd>
                                <dt>Directed By</dt>
                                <dd>{{ details.Director }}</dd>
                                <dt>Written By</dt>
                                <dd>{{ details.Writer }}</dd>
                                <dt>Plot</dt>
                                <dd>{{ details.Plot }}</dd>
                                <dt>Awards</dt>
                                <dd>{{ details.Awards }}</dd>
                                <dt>Run Time</dt>
                                <dd>{{ details.Runtime }}</dd>
                            </dl>
                            <b-button
                                v-if="trailerId !== '' && trailerId !== null"
                                v-b-modal.modal-lg.modal-center
                                variant="warning"
                                size="sm"
                                @click="showTrailer=true"
                            >
                                Watch Trailer
                            </b-button>
                        </b-col>
                    </b-row>

                    <recommendations v-if="similarMovies.length > 0" :items="similarMovies" />
                </b-container>

                <b-modal
                    v-if="trailerId !== '' && trailerId !== null"
                    v-model="showTrailer"
                    size="lg"
                    centered
                    button-size="sm d-none"
                    header-bg-variant="dark text-white"
                    body-bg-variant="dark"
                    footer-bg-variant="dark d-none"
                    :title="'Watch ' + details.Title + ' Trailer'"
                >
                    <iframe
                        v-if="showTrailer"
                        width="100%"
                        height="400"
                        :src="'https://www.youtube.com/embed/' + trailerId"
                        frameborder="0"
                        allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
                        allowfullscreen
                    />
                </b-modal>
            </div>
            <span class="body-bg" :style="posterBg" />
        </template>
        <vue-particles color="#dedede" class="particles" />
    </section>
</template>

<script>
import { mapGetters } from 'vuex'
import Recommendations from '../../components/movie/Recommendations'

export default {
    name: 'Movie',
    components: {
        Recommendations
    },
    data () {
        return {
            showTrailer: false,
            similarMovies: []
        }
    },
    computed: {
        ...mapGetters({
            details: 'movies/movieDetails',
            trailerId: 'movies/trailerId'
        }),
        pageTitle () {
            return (this.details !== null && this.details.Title) ? this.details.Title + ' - Movie Details' : 'Getting movie details...'
        },
        poster () {
            return this.details.Poster.replace('._V1_SX300', '')
        },
        posterBg () {
            return { 'background-image': 'url("' + this.details.Poster.replace('._V1_SX300', '') + '")' }
        }
    },

    async created () {
        await this.$nextTick(async () => {
            const movie = await this.$api.get('', { params: { i: this.$route.params.id } })
            await this.$store.commit('movies/setMovieDetails', movie)
            await this.getTrailer()
            await this.getRecommendations()
        })
    },

    destroyed () {
        this.$store.commit('movies/setMovieDetails', null)
    },

    methods: {
        // Let's call the TheMovieDB API to fetch the trailer
        async getTrailer () {
            const res = await this.$api.get('https://api.themoviedb.org/3/movie/' + this.$route.params.id + '/videos?api_key=f187ec601eb1cd302d1bf2c5d15455ee&language=en-US')
            if (res.results.length > 0) {
                this.$store.commit('movies/storeMovieTrailer', res.results[0].key)
            }
        },
        async getRecommendations () {
            const res = await this.$api.get('https://api.themoviedb.org/3/movie/' + this.$route.params.id + '/similar?api_key=f187ec601eb1cd302d1bf2c5d15455ee&language=en-US')
            if (res.results.length > 0) {
                this.similarMovies = res.results.slice(0, 6)
            }
        }
    },

    head () {
        return {
            title: this.pageTitle
        }
    }
}
</script>

<style lang="scss" scoped>
#movie-details {
    min-height: 100vh;
    margin-top: 6rem;

    .detail-container {
        position: relative;
        z-index: 1;
    }

    .particles {
        position:absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        z-index: 0;
        overflow: hidden;
    }

    .body-bg {
        position: fixed;
        filter: blur(50px);
        background: 50% 50% no-repeat;
        background-size: cover;
        left: calc(10vw * -1);
        top: calc(10vh * -1);
        width: 120%;
        height: 120%;
        z-index: -1;
        &:after {
            content: "";
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.3);
        }
    }
    small {
        font-size: .5em;
    }
}
</style>
