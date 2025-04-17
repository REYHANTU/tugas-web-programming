<template>
  <div id="app">
    <header class="header">
      <div class="logo">🎬 PixelPlay</div>
      <nav class="nav">
        <a href="#" @click.prevent="resetToAll">Home</a>
        <a href="#" @click.prevent="filterToWatchlist">Watchlist</a>
        <a href="#" @click.prevent="alertAbout">About</a>
      </nav>
    </header>

    <div class="controls">
      <div class="search-container">
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Search film..."
          class="search-input"
        />
      </div>

      <div class="filter-container">
        <select v-model="selectedGenre">
          <option value="">All Genre</option>
          <option value="Sci-Fi">Sci-Fi</option>
          <option value="Action">Action</option>
          <option value="Drama">Drama</option>
          <option value="Superhero">Superhero</option>
          <option value="Thriller">Thriller</option>
          <option value="Musical">Musical</option>
          <option value="Psychological Thriller">Psychological Thriller</option>
          <option value="Animation">Animation</option>
          <option value="Comedy">Comedy</option>
          <option value="Biography">Biography</option>
          <option value="Fantasy">Fantasy</option>
        </select>

        <select v-model="selectedYear">
          <option value="">All Years</option>
          <option v-for="year in availableYears" :key="year" :value="year">{{ year }}</option>
        </select>

        <select v-model="sortBy">
          <option value="">Sort</option>
          <option value="a-z">A - Z</option>
          <option value="z-a">Z - A</option>
        </select>
      </div>
    </div>

    <div class="ad-slider" v-if="!isWatchlistView">
      <div class="ad-track">
        <!-- Loop utama -->
        <div class="ad-card card-hover-effect" v-for="(ad, index) in adMovies" :key="index">
          <img :src="ad.image" :alt="'Ad ' + index" />
        </div>
        <!-- Duplikat untuk efek loop -->
        <div class="ad-card card-hover-effect" v-for="(ad, index) in adMovies" :key="'dup-' + index">
          <img :src="ad.image" :alt="'Ad ' + index" />
        </div>
      </div>
    </div>

    <main class="main">
      <div
        class="movie-card"
        v-for="movie in paginatedMovies"
        :key="movie.id"
        @click="showDetail(movie)"
      >
        <img :src="movie.image" :alt="movie.title" class="movie-image" />
        <div class="movie-details">
          <h2 class="movie-title">{{ movie.title }}</h2>
          <p class="movie-genre"><strong>Genre:</strong> {{ movie.genre }}</p>
          <p class="movie-year"><strong>Released:</strong> {{ movie.year }}</p>
          <p class="movie-synopsis">{{ movie.synopsis }}</p>
          <button @click.stop="toggleWatchlist(movie)">
            {{ watchlist.includes(movie.id) ? 'Remove from Watchlist' : 'Add to Watchlist' }}
          </button>
        </div>
      </div>
    </main>

    <div class="pagination">
      <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">Prev</button>
      <span>Page {{ currentPage }} from {{ totalPages }}</span>
      <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">Next</button>
    </div>

    <footer class="footer">
      &copy; 2025 PixelPlay. All rights reserved.
    </footer>

    <!-- Modal Trailer -->
    <div v-if="selectedMovie" class="modal-overlay" @click.self="selectedMovie = null">
      <div class="modal-content">
        <h2>{{ selectedMovie.title }}</h2>
        <iframe
          width="100%"
          height="315"
          :src="selectedMovie.trailer"
          frameborder="0"
          allowfullscreen
        ></iframe>
        <button @click="selectedMovie = null">Close</button>
      </div>
    </div>
  </div>
</template>

<script>
  import './assets/styles.css';
  import { movieList } from './assets/movieData.js';
  import { movieSlider } from './assets/movieSlider.js';

  export default {
    name: 'App',
    data() {
      return {
        movies: movieList,
        searchQuery: '',
        selectedGenre: '',
        selectedYear: '',
        sortBy: '',
        watchlist: [],
        currentPage: 1,
        itemsPerPage: 9,
        selectedMovie: null,
        isWatchlistView: false,
        adMovies: movieSlider,
      };
    },
    computed: {
      availableYears() {
        const years = this.movies.map((movie) => movie.year);
        return [...new Set(years)].sort((a, b) => b - a); // Sort years descending
      },
      filteredMovies() {
        let result = this.movies;

        if (this.isWatchlistView) {
          result = result.filter((movie) => this.watchlist.includes(movie.id));
        }

        if (this.searchQuery) {
          result = result.filter((movie) =>
            movie.title.toLowerCase().includes(this.searchQuery.toLowerCase())
          );
        }

        if (this.selectedGenre) {
          result = result.filter((movie) => movie.genre === this.selectedGenre);
        }

        if (this.selectedYear) {
          result = result.filter((movie) => movie.year == this.selectedYear);
        }

        if (this.sortBy === 'a-z') {
          result = result.sort((a, b) => a.title.localeCompare(b.title));
        } else if (this.sortBy === 'z-a') {
          result = result.sort((a, b) => b.title.localeCompare(a.title));
        }

        return result;
      },
      totalPages() {
        return Math.ceil(this.filteredMovies.length / this.itemsPerPage);
      },
      paginatedMovies() {
        const start = (this.currentPage - 1) * this.itemsPerPage;
        return this.filteredMovies.slice(start, start + this.itemsPerPage);
      },
    },
    methods: {
      changePage(page) {
        if (page >= 1 && page <= this.totalPages) this.currentPage = page;
      },
      toggleWatchlist(movie) {
        if (this.watchlist.includes(movie.id)) {
          this.watchlist = this.watchlist.filter((id) => id !== movie.id);
        } else {
          this.watchlist.push(movie.id);
        }

        // Jika sedang di Watchlist view, update tampilan
        if (this.isWatchlistView) {
          this.currentPage = 1;
        }
      },
      showDetail(movie) {
        this.selectedMovie = movie;
      },
      filterToWatchlist() {
        this.isWatchlistView = true;
        this.currentPage = 1;
        this.searchQuery = '';
        this.selectedGenre = '';
        this.selectedYear = '';
        this.sortBy = '';
      },
      resetToAll() {
        this.isWatchlistView = false;
        this.currentPage = 1;
        this.searchQuery = '';
        this.selectedGenre = '';
        this.selectedYear = '';
        this.sortBy = '';
      },
      alertAbout() {
        alert("Website ini dibuat oleh Reyhan sebagai proyek Vue.js sederhana.\nFitur: search, filter, watchlist, trailer, pagination.");
      }
    }
  };
</script>
