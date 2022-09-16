<template>
<div class="content">
  <header class="header">
      <div class="header__logo">
        <span>
          <LogoBitcoinIcon />
          CryptoFollow
        </span>
      </div>
      <!-- /.header__logo -->
      <div class="header__controls"></div>
      <!-- /.header__controls -->
    </header>
    <!-- /.header -->
    <div class="container">
      <div class="hero">
        <div class="hero__form">
          <div class="hero__form-search">
            <label for="coin">
              Choose and add cryptocoin to track:
            </label>
            <input
              type="text"
              name="coin"
              placeholder="Add coin"
              v-model="ticker"
              v-on:keydown.enter="add"
            >
            <button
              class="btn"
              @click="add"
            >
              Add &nbsp;
              <IosAddCircleIcon />
            </button>
          </div>
          <div class="hero__form-filter">
            <label for="filter">
              Filter coins by name:
            </label>
            <input type="text"
              name="filter"
              placeholder="type name"
              v-model="filter"
            >
            <button class="btn"
              disabled>
              Filter &nbsp;
              <IosAnalyticsIcon />
            </button>
          </div>
          <div class="hero__form-breadcrumbs">
            <button
              class="btn"
              @click="page = page - 1"
              :disabled="page === 1"
            >
              <IosArrowBackIcon />
            </button>
            <button
              class="btn"
              @click="page = page + 1"
              :disabled="hasNextPage === false"
            >
              <IosArrowForwardIcon />
            </button>
          </div>
        </div>
        <template v-if="tickers.length">
          <hr>
          <!-- /.hero__form -->
          <div class="hero__items">
            <div
              class="hero__items-card"
              :class="{
                'picked': sel === coin,
              }"
              v-for="(coin, idx) in filteredTickers()"
              :key="idx"
              @click="select(coin)"
            >
              <h2>{{ coin.name }} - USD</h2>
              <span class="price">{{ coin.price }}</span>
              <button
                class="btn"
                @click.stop="handleDelete(coin)"
              >
              Remove &nbsp;
                <IosTrashIcon />
              </button>
            </div>
          </div>
          <!-- /.hero__cards -->
          <hr>
        </template>

        <div class="hero__graph__section" v-if="sel">
          <div class="hero__graph-controls">
            <h3>{{ sel.name }} - USD</h3>
            <button class="close" @click="sel = null">
              <MdCloseCircleIcon />
            </button>
          </div>
          <div class="hero__graph">
            <div
              v-for="(bar, idx) in normilizeGraph()"
              :key="idx"
              :style="{height: `${bar}%`}"
              class="hero__graph-column"
            ></div>
          </div>
          <!-- /.hero__graph -->
        </div>
      </div>
      <!-- /.hero -->
    </div>
    <!-- /.container -->
</div>
    <footer class="footer">
      <div class="footer__links">
        <a href="#"><span>Github</span> &nbsp;
          <LogoGithubIcon />
        </a>
        <a href="#"><span>LinkedIn</span> &nbsp;
          <LogoLinkedinIcon />
        </a>
        <span>Made by: Yevhen Dovhan</span>
      </div>
    </footer>
</template>

<script>
import LogoBitcoinIcon from 'vue-ionicons/dist/logo-bitcoin.vue';
import IosAddCircleIcon from 'vue-ionicons/dist/ios-add-circle.vue';
import IosAnalyticsIcon from 'vue-ionicons/dist/ios-analytics.vue';
import IosArrowBackIcon from 'vue-ionicons/dist/ios-arrow-back.vue';
import IosArrowForwardIcon from 'vue-ionicons/dist/ios-arrow-forward.vue';
import IosTrashIcon from 'vue-ionicons/dist/ios-trash.vue';
import LogoGithubIcon from 'vue-ionicons/dist/logo-github.vue';
import LogoLinkedinIcon from 'vue-ionicons/dist/logo-linkedin.vue';
import MdCloseCircleIcon from 'vue-ionicons/dist/md-close-circle.vue'

export default {
  name: 'App',
  components: {
    LogoBitcoinIcon,
    IosAddCircleIcon,
    IosAnalyticsIcon,
    IosArrowBackIcon,
    IosArrowForwardIcon,
    IosTrashIcon,
    LogoGithubIcon,
    LogoLinkedinIcon,
    MdCloseCircleIcon,
  },
  data() {
    return {
      ticker: null,
      tickers: [],
      sel: null,
      graph: [],
      page: 1,
      filter: "",
      hasNextPage: true,
    }
  },
  created() {
    const tickersData = localStorage.getItem('crypto-list');
    const windowData = Object.fromEntries(new URL(window.location).searchParams.entries());

    if (windowData.filter) {
      this.filter = windowData.filter;
    }
    if (windowData.page) {
      this.page = windowData.page;
    }

    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach( ticker => {
        this.subscribeToUpdates(ticker.name);
      })
    }
  },
  methods: {

    filteredTickers() {
      const start = (this.page - 1) * 6;
      const end = this.page * 6;
      const filteredTickers = this.tickers.filter(ticker => ticker.name.includes(this.filter));

      this.hasNextPage = filteredTickers.length > end;
      return filteredTickers.slice(start, end);
    },

    subscribeToUpdates(tickerName) {
      setInterval(async () => {
        const f = await fetch(`https://min-api.cryptocompare.com/data/price?fsym=${tickerName}&tsyms=USD&api_key=${process.env.CRYPTO_API}`);
        const data = await f.json();
        this.tickers.find(t => t.name === tickerName).price = data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2);

        if (this.sel?.name === tickerName) {
          this.graph.push(data.USD);
        }
      }, 3000);
    },
    add() {
      const currentTicker = {name: this.ticker, price: "-"}
      this.tickers.push(currentTicker);

      localStorage.setItem("crypto-list", JSON.stringify(this.tickers));
      this.subscribeToUpdates(currentTicker.name);
      this.ticker = '';
      this.filter = '';
    },
    select(ticker) {
      this.sel = ticker;
      this.graph = [];
    },
    handleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter( t => t !== tickerToRemove);
    },
    normilizeGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);
      return this.graph.map(price => (5 + (price - minValue) * 95) / (maxValue - minValue));
    },
    watch: {
      filter() {
        this.page = 1;
        window.history.pushState(null, document.title, `${window.location.pathname}?=filter=${this.filter}&page=${this.page}`);
      },
      page() {
        window.history.pushState(null, document.title, `${window.location.pathname}?=filter=${this.filter}&page=${this.page}`);
      }
    },
  },
}
</script>

<style scoped>
.content {
  flex: 1 0 auto;
}
.hero__graph-controls {
  display: flex;
  justify-content: space-between;
}
.hero__graph-controls button {
  background-color: inherit;
  border: none;
  border-radius: 50%;
  padding: 0.8em;
  position: relative;
  margin-inline-end: 1em;
}
.hero__graph-controls button .ion {
  position: absolute;
  left: 1px;
  top: 0px;
}
.hero__graph-controls button:hover {
  cursor: pointer;
}
</style>
