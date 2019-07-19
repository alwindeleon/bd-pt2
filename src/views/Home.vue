<template>
  <div class="home">
    <button class="button"
            v-if="access_token==null"
            @click="getToken"
    > 
        Login 
    </button>
    <div class="main" v-if="albums.length != 0">
        <h2>School of Rock Related Albums</h2>
        <ul>
            <li v-for="album in albums">
                {{album.name}}
            </li>
        </ul>
        <button class="button-primary" @click="next"> next </button>
    </div>
  </div>
</template>

<script>
import Axios from 'axios';

export default {
  name: 'home',
  data() {
    return {
      access_token: null,
      counter: 0,
      albums: []
    }
  },
  mounted() {
    if( this.$route.params.pathMatch != null ) {
      this.access_token = this.parseAccessToken(this.$route.params.pathMatch);
      this.fetchAlbums().then( response => {
        this.albums = response.data.albums.items;
      })
      .catch( err => {
        console.log(err);
      })
    } 

  },
  methods: {
    fetchAlbums() {
      const instance = Axios.create({
        baseURL: 'https://api.spotify.com/v1/',
        timeout: 1000,
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${this.access_token}`
        }
      });
      return instance.get('search',{ params: {
          q: "School of Rock",
          type: "album",
          limit: 5,
          offset: this.counter*5
      }})
    },
    getToken(){
      window.location.href = "https://accounts.spotify.com/authorize?client_id=0649d4c252144d509de74d43c7478082&redirect_uri=http://blu-spotify.surge.sh&response_type=token"
    },
    parseAccessToken(rawPath) {
        return rawPath.split('&')[0].split('access_token=')[1];
    },
    next() {
        this.counter++;
        this.albums = [];
        this.fetchAlbums().then( response => {
            this.albums = response.data.albums.items;
        })
        .catch( err => {
           console.log(err);
        })
    }
  }
}
</script>