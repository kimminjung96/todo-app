<template>
  <h1>영화목록</h1>
  <div class="row">
    <div v-for="list in movieLists.list" class="card col-6" style="width: 18rem;">
      <img :src="list.Poster" alt="" class="card-img-top" />
      <div class="card-body">
        <h5 class="card-title"> {{ list.Title }}</h5>
        <p class="card-text">{{ list.Plot }}
        </p>
        <a @click="linkMovie(list.imdbID)" class="btn btn-primary">Go somewhere</a>
      </div>
    </div>
  </div>
  <h2>영화상세정보</h2>
  <div class="card" style="width: 18rem;">
    <img :src="movies.list.Poster" alt="" class="card-img-top" />
    <div class="card-body">
      <h5 class="card-title"> {{ movies.list.Title }}</h5>
      <p class="card-text">{{ movies.list.Plot }}
      </p>
      <a href="#" class="btn btn-primary">Go somewhere</a>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref, reactive } from "vue";

export default {
  setup() {
    const URL = `http://www.omdbapi.com/?apikey=7004c9df&s=raw`;
    const API_URL = `http://www.omdbapi.com/?i=tt3896198&apikey=7004c9df`;

    /* const URL = `http://www.omdbapi.com/?apikey=8af1c51&s=raw`;
    const API_URL = `http://www.omdbapi.com/?i=tt6539262&apikey=8af1c51`; */

    const movies = reactive({ list: [] });
    const searchId = ref(null);
    const movieLists = reactive({ list: [] });
    const getMovisList = () => {
      axios
        .get(URL)
        .then(({ data }) => {
          movieLists.list = data.Search
          console.log("🍕", movieLists.list);
        })
    }
    const getMovis = () => {
      axios
        .get(API_URL)
        .then(({ data }) => {
          movies.list = data
        })
    }
    const linkMovie = (id) => {
      searchId.value = id
      console.log(id,searchId.value)
      axios
        .get(`http://www.omdbapi.com/?i=${searchId.value}&apikey=8af1c51`)
        .then((res) => {
          res
        })
    }
    getMovisList();
    getMovis();
    return {
      movies, movieLists, linkMovie
    }
  }
}
</script>

<style></style>